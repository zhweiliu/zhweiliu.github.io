---
title: Migrate Google Cloud Functions to Kubernetes
date: '2022-01-23T05:59:52.712Z'
categories: ['Google Cloud Platform', 'Data Engineering']
keywords: ['Google Cloud Platform', 'GCP', 'ETL Pipeline']
---

在 [GCP Billing Analytics](https://medium.com/@zhweiliu/gcp-billing-analytics-b1d1edf6ad38?source=your_stories_page----------------------------------------) 中提到過關於 Cloud Functions 的計費超乎預期，進一步分析開發的使用習慣後，也找出部分功能應該將其從 Cloud Functions 搬遷至基於 GCE instances 的服務上，以達到節費的期望。

在原先的設計中，我們將 Cloud Functions 作為 ETL data flow 的其中一個環節，透過 Pub/Sub trigger Cloud Functions 的方式使其運作；考慮到 [Pub/Sub subscriber push/pull](https://cloud.google.com/pubsub/docs/subscriber#push_pull) 的 Ack 等待時間有著最長 600 秒的限制，我將這部分需要搬遷的 Cloud Functions 大致分為兩種需求

1.  `靜態資料源`: 在提取資料時，可預期資料是存在且可被存取的
2.  `動態資料源`: 可能發生資料不存在，或者是無法存取的情況

本篇文章是記錄

*   [用 `Kubernetes Pod` 替代 `Cloud Function` 環節以處理`動態資料源`的方法](#1234)
*   [`Google Kubernetes Engine: Ingress & Service`](#dee1)
*   [`ASGI 與FastAPI`](#fa5e)
*   [`Dockerize & Deployment`](#b458)

`靜態資料源`的處理方案 > [`Migrate Google Cloud Functions to Airflow`](https://medium.com/@zhweiliu/migrate-google-cloud-functions-to-airflow-bde12ffec8df?source=your_stories_page----------------------------------------)

### `Design Change`

![](/images/normal/migrate-google-cloud-functions-to-kubernetes/image_0.png)
Figure 1 是一個常見的使用案例，我將 Cloud Function 的執行邏輯簡略為 4 個部份來進行描述，即: 等待 Request (Accept Request) 、 處理邏輯 (Process)、產出結果 (Result) ，以及回復 Ack (Response HTTP Status Code)

Process 的區塊中，若需要向外部資料源提出存取請求，如: 3rd-party API 、爬蟲、網路磁碟機等，獲取相關的資訊後才能繼續進行處理的工作，在本篇文章中則以`動態資料源`來稱呼這些外部資料源

> 對於 Runtime 時可能遭遇錯誤的資料源，可能遇到請求被拒絕(Reject)，如: 403、404或者5系列的錯誤代碼，或是遇到請求的資源本身不存在。

### Google Kubernetes Engine: Ingress & Service

![](/images/normal/migrate-google-cloud-functions-to-kubernetes/image_1.png)
Figure 2 使用 Kubernetes Pod 替代 Cloud Function ， 因團隊先前已採用 Google Kubernetes Engine (GKE) 進行容器化的部署，這邊也就延續團隊成果。

我也將 Pub/Sub 的模式從 trigger 更改為 [Push Message](https://cloud.google.com/pubsub/docs/push) : 當 Pub/Sub Subscriber Queue 存在訊息時， Subscriber 會推送 Message 到設定好的 Webhook URL，並且遵循 Ack 等待時間有著最長 600 秒的限制。

關於 Deployment 的部分會在稍後提到，這邊先討論 Ingress 和 Service 的設置

Service Type: [NodePort](https://kubernetes.io/docs/concepts/services-networking/service/#type-nodeport)

```
apiVersion: v1kind: Servicemetadata:  name: my-servicespec:  type: NodePort  selector:    app: MyApp  ports:      # By default and for convenience, the `targetPort` is set to the same value as the `port` field.    - port: 80      targetPort: 80      # Optional field      # By default and for convenience, the Kubernetes control plane will allocate a port from a range (default: 30000-32767)      nodePort: 30080
```

[Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/#ingress-rules)

`apiVersion`: networking.k8s.io/v1  
`kind`: Ingress  
`metadata`:  
  `name`: ingress-service-backend  
  `annotations`:  
    ingress.gcp.kubernetes.io/pre-shared-cert: "k8s-example-com"  
    kubernetes.io/ingress.allow-http: "false"  
    kubernetes.io/ingress.global-static-ip-name: k8s-example-com  
`spec`:  
  `defaultBackend`:  
    `service`:  
      `name`: my-services  
      `port`:  
        `number`: 80  
  `rules`:  
    - `host`: k8s.example.com  
      `http`:  
        `paths`:  
          - `path`: /my-service  
            `pathType`: Prefix  
            `backend`:  
              `service`:  
                `name`: my-service  
                `port`:  
                  `number`: 80

這樣便能將 Ingress 和 Service 設置完成，Ingress 和 Service 需要在同一個 namespace 。

### ASGI & FastAPI

考量到團隊開發大部份依賴 Python framework，因此在替代 Cloud Function HTTP Server 的選擇上，最後我採用了基於 ASGI (Asynchronous Server Gateway Interface) 的 [FastAPI](https://fastapi.tiangolo.com/id/) ，以應付團隊中除了 Pub/Sub 之外的需求。

對於 WSGI 和 ASGI 的比較，我覺得這篇博客 [WSGI与ASGI的区别与联系](https://blog.csdn.net/huayunhualuo/article/details/106007545) 說的很清楚，推薦大家可以看一下。

FastAPI 的文件中也詳細提供了[製作 Container Image](https://fastapi.tiangolo.com/id/deployment/docker/#build-a-docker-image-for-fastapi) 的方法，同時也提到了關於[部署在 Kubernetes 上的注意事項](https://fastapi.tiangolo.com/id/deployment/docker/#replication-number-of-processes)，有一份詳細、容易使用的官方文件，也是我選擇 FastAPI 的原因之一，並且 FastAPI 也內建了 [Swagger UI](https://fastapi.tiangolo.com/id/deployment/docker/#interactive-api-docs) 和 [ReDoc](https://fastapi.tiangolo.com/id/deployment/docker/#alternative-api-docs) 兩種文件模式，這也是一個加分大項。

### Dockerize & Deployment

`Dockerfile`

依據 FastAPI 文件提供 [Dockerfile](https://fastapi.tiangolo.com/id/deployment/docker/#dockerfile) 撰寫即可，需注意在 `uvicorn` 的 command加上 `--proxy-headers` 。

`FROM` python:3.8

`WORKDIR` /

`COPY` ./requirements.txt /requirements.txt

`RUN` pip install --no-cache-dir --upgrade -r /requirements.txt

`COPY` ./ /

`CMD` ["uvicorn", "main:app", "--proxy-headers", "--host", "0.0.0.0", "--port", "80"]

依需求更改 Dockerfile 時需要注意 Docker Build Cache，由於 Docker Build Image 時會一層一層的往上迭代(每一行指令就是一層)， 而每一次 Build Image 都會檢查與上一次的差異，並從影響差異的 `最低層` 重新迭代，如: 當 requirements.txt 內容有所變更時，即便 source code 沒有改變，該次的 Docker Build 也會從 `COPY` ./requirements.txt /requirements.txt` 開始從新迭代。

`Main.py`

在 main.py 提供 domain host 之後的完整 URL path ，讓 app 的 route 可以找到對應的端口，並提供 `/my-service/health` 給 Load Balancer 進行 health check。

`from` typing `import` Optional, Dict  
`from` fastapi `import` (FastAPI, status)  
`from` fastapi.encoders `import` jsonable_encoder  
`from` pydantic import `BaseModel`

`class Message(BaseModel):`  
    `attrs`: Optional[Dict] = None  
    `data`: str  
    `message_id`: str  
    `publish_time`: str

`class PubSubMessage(BaseModel):`  
    `message`: Message  
    `subscription`: str

`app` = FastAPI()

[`@app`](http://twitter.com/app "Twitter profile for @app")`.get('/', status_code=status.HTTP_200_OK)`  
`def home`():  
    pass

[`@app`](http://twitter.com/app "Twitter profile for @app")`.get('/my-service/health', status_code=status.HTTP_200_OK)`  
`def health`():  
    pass

`@app.post('/my-service/subscriber-webhook', status_code=status.HTTP_200_OK)`  
`def subscriber_webhook`(`message`: `PubSubMessage`):  
    `message_data`: Dict = jsonable_encoder(message)  
    `return` message_data

`Deployment`

依據 [Kubertenes 官方提供的模板](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#creating-a-deployment)撰寫，再依需求進行更改即可。

`apiVersion`: apps/v1  
`kind`: Deployment  
`metadata`:  
  `name`: subscriber-webhook-deployment  
  `labels`:  
    `app`: subscriber-webhook  
`spec`:  
  `replicas`: 3  
  `selector`:  
    `matchLabels`:  
      `app`: subscriber-webhook  
  `template`:  
    `metadata`:  
      `labels`:  
        `app`: subscriber-webhook  
    `spec`:  
      `containers`:  
      - `name`: subscriber-webhook  
        `image`: {REPLACE_YOUR_REGISTRY}/subscriber-webhook:1.0  
        `ports`:  
        - `containerPort`: 80

可視需要加入 `[`readinessProbe`](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/#define-a-tcp-liveness-probe)` [或](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/#define-a-tcp-liveness-probe) `[`livenessProbe`](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/#define-a-tcp-liveness-probe)`

如果有 Autoscaling 的需求，參考 [Horizontal Pod Autoscaling (HPA)](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/#default-behavior) 與 [範例](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/#autoscaling-on-multiple-metrics-and-custom-metrics) 修改即可。