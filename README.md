# MSDA K8S iAppLX

This iApp is an example of accessing tmsh.  The iApp itself is very simple - it manages the members of a pool.

## Build (requires rpmbuild)

    $ npm run build

Build output is an RPM package.

## Using IAppLX from BIG-IP UI
If you are using BIG-IP, install f5-iapplx-msda-k8s RPM package using iApps->Package Management LX->Import screen. 
To create an application, use iApps-> Templates LX -> Application Services -> Applications LX -> Create screen. 
Default IApp LX UI will be rendered based on the input properties specified in basic pool IAppLX.


## Create the base64 encodng certificate for authentication

1. check the certificate of your account in kube config, for example

```
root@ubuntu:~# cat .kube/config 
apiVersion: v1
clusters:
- cluster:
    certificate-authority: /root/.minikube/ca.crt
    extensions:
    - extension:
        last-update: Fri, 06 May 2022 12:30:12 UTC
        provider: minikube.sigs.k8s.io
        version: v1.25.2
      name: cluster_info
    server: https://10.1.1.160:8443
  name: minikube
contexts:
- context:
    cluster: minikube
    extensions:
    - extension:
        last-update: Fri, 06 May 2022 12:30:12 UTC
        provider: minikube.sigs.k8s.io
        version: v1.25.2
      name: context_info
    namespace: default
    user: minikube
  name: minikube
current-context: minikube
kind: Config
preferences: {}
users:
- name: minikube
  user:
    client-certificate: /root/.minikube/profiles/minikube/client.crt
    client-key: /root/.minikube/profiles/minikube/client.key
root@ubuntu:~# 
```
In this example, you will have 

```
certificate-authority: /root/.minikube/ca.crt
- name: minikube
  user:
    client-certificate: /root/.minikube/profiles/minikube/client.crt
    client-key: /root/.minikube/profiles/minikube/client.key
```

2. Convert the certificates and key into base64 encoding


```
root@ubuntu:~# 
root@ubuntu:~# cat /root/.minikube/profiles/minikube/client.key | base64 -w 0 > clientkeyb64
root@ubuntu:~# cat /root/.minikube/profiles/minikube/client.crt | base64 -w 0 > clientcrtb64
root@ubuntu:~# cat /root/.minikube/ca.crt |  base64 -w 0 > cab64
root@ubuntu:~# 
root@ubuntu:~# 
root@ubuntu:~# ls -l
total 39812
-rw-r--r--  1 root root     1484 May  6 15:08 cab64
-rw-r--r--  1 root root        0 May  6 15:07 clientcertb64
-rw-r--r--  1 root root     1532 May  6 15:07 clientcrtb64
-rw-r--r--  1 root root     2240 May  6 15:07 clientkeyb64
drwxr-xr-x 11 root root     4096 Dec 10 01:21 etcdlabs
-rw-r--r--  1 root root 24363252 May  6 11:28 minikube_latest_amd64.deb
-rw-r--r--  1 root root 16360927 May  6 11:14 minikube-latest.x86_64.rpm
drwxr-xr-x  8 root root     4096 May  6 12:43 MSDA-Demo
drwxr-xr-x  7 root root     4096 May  4 07:18 nacos-docker
drwxr-xr-x 30 root root     4096 Dec 10 12:03 NGINX-Demos
drwxr-xr-x  6 root root     4096 Jan  3 05:20 odyssey-lift-off-part1
drwxr-xr-x  3 root root     4096 Jan  3 07:46 snap
root@ubuntu:~# 

```

3. save the base64 files, input them if the template or application LX form.
4. If you want to create a new account for bigip, please refer to the manual : https://kubernetes.io/docs/reference/access-authn-authz/certificate-signing-requests/#normal-user .



## Using IAppLX through restful to configure BIG-IP

Configure through RESTful API with the IAppLX package. Pass in the remote BIG-IP to be trusted when starting REST container as environment variable.

Create an Application LX block with all inputProperties as shown below.
Save the JSON to block.json and use it in the curl call. Refer to the clouddoc link for more detail: https://clouddocs.f5.com/products/iapp/iapp-lx/tmos-14_0/iapplx_ops_tutorials/creating_iappslx_with_rest.html 

```json
{
  "name": "msdak8s",
  "inputProperties": [
    {
      "id": "k8sEndpoint",
      "type": "STRING",
      "value": "https://1.1.1.1:8443",
      "metaData": {
        "description": "k8s endpoint list",
        "displayName": "k8s endpoints",
        "isRequired": true
      }
    },
    {
      "id": "authenticationCert",
      "type": "JSON",
      "value": {
        "clientCert": "LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSURJVENDQWdtZ0F3SUJBZ0lCQWpBTkJna3Foa2lHOXcwQkFRc0ZBREFWTVJNd0VRWURWUVFERXdwdGFXNXAKYTNWaVpVTkJNQjRYRFRJeU1EVXdOVEV5TWpBd05Gb1hEVEkxTURVd05URXlNakF3TkZvd01URVhNQlVHQTFVRQpDaE1PYzNsemRHVnRPbTFoYzNSbGNuTXhGakFVQmdOVkJBTVREVzFwYm1scmRXSmxMWFZ6WlhJd2dnRWlNQTBHCkNTcUdTSWIzRFFFQkFRVUFBNElCRHdBd2dnRUtBb0lCQVFDMmpLaWRPRDErbHlBZENOcjAxREJsekNNYWZkbysKU29NUXpIZlU5aWJCK1BzbFRVdDZ5ZGdmV1JySW80aFNEZFBmZEJrWUZhT3BjOHNaMk5sMjZ6MS8ra29vYTA3UgprUFZyUDJCdjBKaTl3c3ErQjNncUl2eFM5cDh5QnNWczZ6OTdjalgwaUY4QnV5Wk5mMjRSNkhKU2FkUDhOMHhKClBzbkdiRkVudGg3NW83S0orN3V2T1E1cFgxNkg5WmNVUy9RTlk2eUVTMThYQWgweVdRUFpmVVRSaXVOWnZOTmUKRzUyaUd2aDZ5V3E0SXdwMmNTTUxON2ZwSFdwcGlsVjR0Z3RYSXk4TVZRbDUzWThxei9JcHlGQk5ORUZOQ3hqYwpmV3A5WE1PWFArWHk2ZHgrWGU2OGx5T1lZS1JnZVpPQUU3bWxoYVJXUkt0QkxZZGwwd1ZJMGhaZkFnTUJBQUdqCllEQmVNQTRHQTFVZER3RUIvd1FFQXdJRm9EQWRCZ05WSFNVRUZqQVVCZ2dyQmdFRkJRY0RBUVlJS3dZQkJRVUgKQXdJd0RBWURWUjBUQVFIL0JBSXdBREFmQmdOVkhTTUVHREFXZ0JRMm5XU2h2Zm51VG5LSmpEdjcxMW9ROVdzLwpXekFOQmdrcWhraUc5dzBCQVFzRkFBT0NBUUVBQmhCWnRVdzQwTHc3ZTVjTDNPUCtmaTQ0eDBjYVhXZzgrRFJQClR3WjF3MzMrS2xWRFpDRGFUZXN4b1JwVnJnTVhZU0daVHB5MDNoVlhtS0RJSUx3L3kreDBYRGgrRTN2bGxOS3UKZ1JkeTZCZ3RDMjRKeWM3cGlwSlFta1Y5ajlab0RHdGxzVHgyOGxyNm5GcE5QZ1ZQYzJiN0pGL29pQ3VKZnpUMQp4M0cvTjdJelNoU2p1b3JYTlU5R05uaVplSHpEdDB3V01ObnJXOHU2bWtOZ25JRGNrL2xhRU5FenMvejZWcXpQClQvZzJzUzNjcjg0aitLQlVZTUJNeVpmakVuUlluaDN5ZU1ia0pLUHpkQTJBZmVCWU9ycEtzNEFyU05MT0ZtT1oKazFNZzVRZ3dZQW92ODZxR0I5czdlV2x4cEQ5NVR1MVBpWlducWV3a1ZuZW5nZXlmQ2c9PQotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg==",
        "clientKey": "LS0tLS1CRUdJTiBSU0EgUFJJVkFURSBLRVktLS0tLQpNSUlFcEFJQkFBS0NBUUVBdG95b25UZzlmcGNnSFFqYTlOUXdaY3dqR24zYVBrcURFTXgzMVBZbXdmajdKVTFMCmVzbllIMWtheUtPSVVnM1QzM1FaR0JXanFYUExHZGpaZHVzOWYvcEtLR3RPMFpEMWF6OWdiOUNZdmNMS3ZnZDQKS2lMOFV2YWZNZ2JGYk9zL2UzSTE5SWhmQWJzbVRYOXVFZWh5VW1uVC9EZE1TVDdKeG14Uko3WWUrYU95aWZ1NwpyemtPYVY5ZWgvV1hGRXYwRFdPc2hFdGZGd0lkTWxrRDJYMUUwWXJqV2J6VFhodWRvaHI0ZXNscXVDTUtkbkVqCkN6ZTM2UjFxYVlwVmVMWUxWeU12REZVSmVkMlBLcy95S2NoUVRUUkJUUXNZM0gxcWZWekRsei9sOHVuY2ZsM3UKdkpjam1HQ2tZSG1UZ0JPNXBZV2tWa1NyUVMySFpkTUZTTklXWHdJREFRQUJBb0lCQUYxd3ZsWkxsVjZZNkwwego3Uy9vOVNVR1N1bWloZlhnbWhvZEx6RjVGZm13QW8zamRNRlRWQ2NucXdnTWZSalRMeUp3QVBCTkUwc0hsR3lVCmpTdkwyZDBLTnE5ZHppaUROTHhDNHBBWmpEV0Y0ZFZIYVlEWUM2UkR6TlVFbGtYY1hOQkpjOGpKalNnTHJkMTUKWHRRWDBYelI0c3AxVzcwYVFKb3FrNWZxSnd6TW1wb1dWT2xOeDM4Q2F3NGlFcjUySHljVXZaeXRPTUQxNERwcgphcCtoc05kUnFqM1FNQWR0T1Z6NHhBWjdOV1ZlVm1LbXpzOFRYVEpnaHVCYzRVeUtHYXZ0K2QzNVpaaUtnSnBWClVRSWxud21rd2lHakxIcUZMVXA3R1Y2U1hITmJRdFhpei9WTVAxTVRUSjlqcWpPcGxXMWozc3NJWlpIOU1aZHAKYXl2VEVKRUNnWUVBM0pVNUZyR1VDd3RJL2ZsU2dFQ0NLWVhSd1pXckczQS80L082dC9ucFpEclZIbXlVdXRMOQpFQ0ZaamRPdHF2UHoxdzgrV2tKa25xQW92WkY5TmIxVWFqZHlRVFNsbmNzSXdlM09mcjFxQTVuZXlVRnpGSTRoCkg2SmU3RDVWUHlZOUVtVUg1dWJDcHdyL0dIS2JJYUdQNW1mVzNaQVA2T2s0WEFMaGpibkErU01DZ1lFQTA5d2UKakdQbUhnNUxpVkhHd0ZDTnVHcytUY01YTmRKelpmb2pQYnR1OENvRlllZ09QZ0E0UzNYSDNYYkhDdDZMZ3RLQQpNMUdIMWpGRk1QTlA2bkIyTTZqM1FzT2UyVVRnbXpOM2xxOGZUVDJGOGRwd0xqUlBVaFBINDVOQW5qMUZBSXdaCk1lbmFYNGNtTFdNaEpVVnV0V2tNbUN1cHRraDlwcXhoRituc1o1VUNnWUJHUDBPQ0JhVjI2dTRnNjdDcFpXSE8KWlc2S2J1YWlBMXBsZHU0a3Z2TGoxNVNkYnNqaXdtU1RLWHZDbmdIMXFtRWlRUm1EVnhlQ0tORXdwYyt4T0kxVQpram5ScURtQ0NmSE5DTFcxU1E4ay9IQ2x1VEV6LzV0dTNwL0tMb09wYTcwUlNabDlvRW1uTnVwTVY5c3RsNjBqCkhEaWlNTW5RUzgyR0IramE0S2dpN3dLQmdRQ0djWmg5Tk9RU1RMWUl0WGx3RDI1d0NyWmwrSmpoRWVVallNSSsKYVpSMEdlYUNoQldOcU93UWp2Uy9pS0cxTnhiSGRUZmYyU3hmYzdMWjVuM2ZZM0RQUmJscmgrSmxOSDFvWUJmUwo2dHp0VWs2TzlUVGRUVnJNMWpxeUkzOE5MQXArMTJraHNLcGdsczVXWFNMcW1RNHhWekdqMjRsK1lMQkVOZjRECmcvSCtwUUtCZ1FDbk0wMjFTdCtCS0FXVjRSZWRHNDE1eUE4bENjSnJQaHFFZ05NNnNuK010WTNSWWs3QitIT1cKQStvRHg0dnZvbWlweXBCV3p3K0JGeTh5Q1cxUVpCT2lER2FlMHVESW1PQ3hZeGhrR2d3OW9nTUl3emNFdEZ0dgpNYU9CRXIvZlRoU1lFc1RBR043bldvblJjSytaMUkvLzgvdHRyK1dhU1pjM0xRNkdtZHRHNFE9PQotLS0tLUVORCBSU0EgUFJJVkFURSBLRVktLS0tLQo=",
        "caCert": "LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSURCakNDQWU2Z0F3SUJBZ0lCQVRBTkJna3Foa2lHOXcwQkFRc0ZBREFWTVJNd0VRWURWUVFERXdwdGFXNXAKYTNWaVpVTkJNQjRYRFRJeU1EVXdOVEV5TWpBd05Gb1hEVE15TURVd016RXlNakF3TkZvd0ZURVRNQkVHQTFVRQpBeE1LYldsdWFXdDFZbVZEUVRDQ0FTSXdEUVlKS29aSWh2Y05BUUVCQlFBRGdnRVBBRENDQVFvQ2dnRUJBTDdvCk5uWktxL3ArVWNPQ1RjZkd2UXNaeFlVOEw2TWhIWnRuRHppSGFsZTFMOHI1TW5uNTk5M1pUcHYrUmtlSVF1QlkKTkl1YU9wWUg2ZE9Mc1BjZlVaWURXV2U0S2JsTzlTVG5rbUpuMGVVRjNZejBVbC9DdW41bTZTRERLOUdpQXk0egpBM05qYlBMcTlyQTJYaks4RGJ2UUplSUZlaWZraEFqb1dlaXplRUJzRjFnM0tydlg1WVliTHFJenFBRXRXR2VGCkhTb2VOYnIwa0JleHlRN2RFbE5ONW1pRlltbFdWb3BpTThXOUdTdkJyVkE4VlZpRDVxMDBsWkZyM0dWZXJzSHIKNHJBUTlNYmNKQit1Wjl3NjUyQXJsWHpVNThTblVFcHBEM1ZRSW4reWE3SmlsYldVekVLeE11aitOWFVrNGhKcwpEdEtFMnRKN2tRKzVEY0FMd2dNQ0F3RUFBYU5oTUY4d0RnWURWUjBQQVFIL0JBUURBZ0trTUIwR0ExVWRKUVFXCk1CUUdDQ3NHQVFVRkJ3TUNCZ2dyQmdFRkJRY0RBVEFQQmdOVkhSTUJBZjhFQlRBREFRSC9NQjBHQTFVZERnUVcKQkJRMm5XU2h2Zm51VG5LSmpEdjcxMW9ROVdzL1d6QU5CZ2txaGtpRzl3MEJBUXNGQUFPQ0FRRUF1dGJ6VENaMgpwR1pmNnR6dE02aHN1ZzlGaVdPazhwM1NYc0RXMm9mM2ZMdk1uSEd4SUM4endOZHNxQVM2dDN3NGJwRmFLRjR4CjRvQWt3TjlST2RmSmJnanJpU0xkTVpRU2ZmNGJ0TUlkcURNaHpkcXU4aUtOZUtLZzZBUGJIWVM4QlBXeFppOWUKTVFoSDVKdHd5RTBlamR0T0hvcU44TW9qRU1rQjkrRlhuTDlzS2lCdUw1SVc0d09uc3NtbWNzKzVRTXZkRTA4RgpObWJjL1hweGxEeE03TDROdGc0S1h2QzlteE9tWFBVUDdRL0htdUFjK1ViOFkvZVREWWNVZUpoS2RMOXpxYzIxCjd0MUl5MWJnalEvbE11eTJObjYzRWgrckZTV1cxRjN1dThkY0U3SGs4ZWZaWkU1VnpvK1BMbGRCNTBpRWt2dWkKcGFyaStXUndaclY4d2c9PQotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg=="
      },
      "metaData": {
        "description": "k8s lient authentication certificate in base64 encoding",
        "displayName": "k8s client certificate",
        "isRequired": true
      }
    },
        {
      "id": "nameSpace",
      "type": "STRING",
      "value": "default",
      "metaData": {
        "description": "Namespace in kubernetes",
        "displayName": "Service Name in registry",
        "isRequired": true
      }
    },
    {
      "id": "serviceName",
      "type": "STRING",
      "value": "msda",
      "metaData": {
        "description": "Service name to be exposed",
        "displayName": "Service Name in registry",
        "isRequired": true
      }
    },
    {
      "id": "poolName",
      "type": "STRING",
      "value": "/Common/k8sSamplePool",
      "metaData": {
        "description": "Pool Name to be created",
        "displayName": "BIG-IP Pool Name",
        "isRequired": true
      }
    },
    {
      "id": "poolType",
      "type": "STRING",
      "value": "round-robin",
      "metaData": {
        "description": "load-balancing-mode",
        "displayName": "Load Balancing Mode",
        "isRequired": true,
        "uiType": "dropdown",
        "uiHints": {
          "list": {
            "dataList": [
              "round-robin",
              "least-connections-member",
              "least-connections-node"
            ]
          }
        }
      }
    },
    {
      "id": "healthMonitor",
      "type": "STRING",
      "value": "none",
      "metaData": {
        "description": "Health Monitor",
        "displayName": "Health Monitor",
        "isRequired": true,
        "uiType": "dropdown",
        "uiHints": {
          "list": {
            "dataList": [
              "tcp",
              "udp",
              "http",
              "none"
            ]
          }
        }
      }
    }
  ],
  "dataProperties": [
    {
      "id": "pollInterval",
      "type": "NUMBER",
      "value": 30,
      "metaData": {
        "description": "Interval of polling from BIG-IP to registry, 30s by default.",
        "displayName": "Polling Invertal",
        "isRequired": false
      }
    }
  ],
  "configurationProcessorReference": {
    "link": "https://localhost/mgmt/shared/iapp/processors/msdak8sConfig"
  },
  "audit": {
    "intervalSeconds": 0,
    "policy": "NOTIFY_ONLY"
  },
  "configProcessorTimeoutSeconds": 30,
  "statsProcessorTimeoutSeconds": 15,
  "configProcessorAffinity": {
    "processorPolicy": "LOAD_BALANCED",
    "affinityProcessorReference": {
      "link": "https://localhost/mgmt/shared/iapp/processors/affinity/load-balanced"
    }
  },
  "state": "TEMPLATE"
}
```

Post the block REST container using curl. Note you need to be running REST container for this step
and it needs to listening at port 8433
```bash
curl -sk -X POST -d @block.json https://localhost:8443/mgmt/shared/iapp/blocks
```
