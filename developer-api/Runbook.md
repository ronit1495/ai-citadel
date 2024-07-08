# I/O Exchange Developer API 5XX Errors

I/O Exchange Developer API 5XX Errors

## About Developer API

- Developer API microservice is one of the very important microservices in ARS. 
- It's very cruicial to monitor this service as it holds entire lifecycle of developers app developemnt which includes creating assets, uploading assets, etc.
- Util Service and CC Admin service also uses developer API and are dependent service.
- Developer API stores develoepr's application data on MongoDB via CC Admin service, CC admin service helps in avoiding direct calling of MongoDB from Developer API.
- Assets are uploaded on S3 temp Bucket, it's then scaned by Util Service and Util service stores the scan result data in MySql.
- Developer API uses AWS IAM credentials to connect to S3 for fetc and upload operations.
- MyExchange APIs also consume these APIs.

## Alert

- Whenever splunk gateway logs encounters 5XX errors (inlcudes `500`, `502`, `503` & `504`), it triggers slack alert in the #io-exch-monitoring-sev2 channel.
- This alert will be raised when error rate is more than `0.02%`
- Please check the Error Logs
  
## Causes

- `500` indicates error handling is done incorrectly.
- `502` indicates request from gateway did not reach application pods.
- `503` indicates service is unavailable due scalability issues or server startup issues.
- `504` indicates the gateway request being timed out of your service.

## Investigation

- Check if you can locate alert in the above reports/splunk queries with appropriate date & time.
- Check for `reqId` or `x-request-id` in the gateway logs or ask CCES team if there are failing requests.
- If the status code is `500` then try locating the same reqId in splunk internal (application) logs for detailed information.

## Triage

- If the status code is `500` and you think that error handling is not done, then please log bug in [JIRA-XCHG]
- If the status code is `502` and if you can't find any logs related to it then please log bug in [JIRA-EON]. Eg: [Sample EON Ticket]
- If the status code is `503` then please login to kubernetes cluster with appropriate namespace using below commands:
    ```sh
      export KUBECONFIG=kubeconfig.yaml
      kubectl config use-context ethos61prodva7
      kubectl config set-context --current --namespace=ns-team-ethos61prodva7-exchange-prod-b
      kubectl get pods
    ```
  If you observe developer api pod status as CrashLoopBackOff then it might deployment or server startup issue due to invalid configmap/secret values
    ```sh
      xchg-developer-api-67fd757666-4gzb8   1/2     CrashLoopBackOff   7          10m
    ```
  If you observe entitlement pod restarts more than once (like below) then it might be scalability issue.
    ```sh
      xchg-developer-api-69fbb894cf-6lrdm   2/2     Running       19         25d
      xchg-developer-api-69fbb894cf-hff68   2/2     Running       2          14d
      xchg-developer-api-b9c99fffd-hk94f    0/2     Terminating   0          11m
    ```
  To see more details about scalability issue, you can refer below commands:
    ```sh
      watch -n 0 kubectl get  hpa
      xchg-developer-api   Deployment/xchg-developer-api   12%/80%   2         10        2          2d1h
      xchg-developer-api   Deployment/xchg-developer-api   66%/80%   2         10        2          2d1h
    ```
- If the status code is `504` then it might due to following reasons:
  - Slow running mongoDB queries: can be fixed by adding indexes or refactoring the query.
  - Data issue (user with too many entitlements): Checking developer API collection will help to find the developer data.
  - Redis timeout: Check splunk internal logs and azure logs.
  - Gateway timeout in IMS/IPAAS apis: Checking total time taken in logs for all external apis calls when they go beyond given threshold time. 
  - Latency check in Azure port: You can check latency in all components like [CosmosDB], [Redis] & [CDN].
