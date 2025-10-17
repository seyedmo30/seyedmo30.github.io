
+ collections

میتونیم یه کالکشن بسازیم و تمامی end point  هارو تو اون بندازیم

هر جایی می تونیم ازش استفاده کنمی ، در variable ، در اسکریپت کد و در url

```

{{mobileNo}}


{{baseUrl}}/oauth/token

```
+ variables

می توان این را به صورت دستی یا نتایج endpoint ها را در این ریخت

+ tests 
می تونیم بعد از درخواست و گرفتن پاسخ ، با اسکریپت نویسی  ، متغییر ها رو مقدار دهی کنیم

```
const responseJson = pm.response.json()
pm.collectionVariables.set("Authorization","Bearer "+responseJson.access_token)
```

+ pre request script

می تونیم قبل از درخواست ، اسکریپت اجرا کنیم

```
var uuid = require('uuid')

const currentDate = new Date();
pm.collectionVariables.set("startDate",currentDate.toISOString())
const expirationDate = new Date(currentDate.setMonth(currentDate.getMonth()+6));
pm.collectionVariables.set("expirationDate",expirationDate.toISOString())
const traceID = uuid.v4()
pm.collectionVariables.set("traceId",traceID)
```




### EXAMPLE

کد زیر رو می توان با فرمت جیسون ایمپورت کرد

```

{
	"info": {
		"_postman_id": "3cbb646c-051e-4114-9ca5-39e905e7889f",
		"name": "payman",
		"schema": "https://schema.getpostman.com/json/collection/v2.1.0/collection.json",
		"_exporter_id": "31331385"
	},
	"item": [
		{
			"name": "phase 1",
			"item": [
				{
					"name": "login",
					"event": [
						{
							"listen": "test",
							"script": {
								"exec": [
									"const responseJson = pm.response.json()",
									"pm.collectionVariables.set(\"Authorization\",\"Bearer \"+responseJson.access_token)"
								],
								"type": "text/javascript"
							}
						}
					],
					"request": {
						"method": "POST",
						"header": [
							{
								"key": "App-Key",
								"value": "{{AppKey}}"
							},
							{
								"key": "Content-Type",
								"value": "application/x-www-form-urlencoded"
							},
							{
								"key": "Accept",
								"value": "application/json"
							},
							{
								"key": "Client-Ip-Address",
								"value": "127.0.0.1"
							},
							{
								"key": "Client-Platform-Type",
								"value": "WEB"
							},
							{
								"key": "Client-Device-Id",
								"value": "127.0.0.1"
							},
							{
								"key": "Client-User-Id",
								"value": "{{mobileNo}}"
							},
							{
								"key": "Client-User-Agent",
								"value": "firefox5.0"
							},
							{
								"key": "Cookie",
								"value": "cookiesession1=678A8C34D3B6F498C549C66E39390D07",
								"type": "text"
							}
						],
						"body": {
							"mode": "urlencoded",
							"urlencoded": [
								{
									"key": "client_id",
									"value": "{{clientId}}",
									"type": "text"
								},
								{
									"key": "client_secret",
									"value": "{{clientSecret}}",
									"type": "text"
								},
								{
									"key": "grant_type",
									"value": "client_credentials",
									"type": "text"
								}
							]
						},
						"url": {
							"raw": "{{baseUrl}}/oauth/token",
							"host": [
								"{{baseUrl}}"
							],
							"path": [
								"oauth",
								"token"
							]
						}
					},
					"response": [
						{
							"name": "login",
							"originalRequest": {
								"method": "POST",
								"header": [
									{
										"key": "App-Key",
										"value": "{{AppKey}}"
									},
									{
										"key": "Content-Type",
										"value": "application/x-www-form-urlencoded"
									},
									{
										"key": "Accept",
										"value": "application/json"
									},
									{
										"key": "Client-Ip-Address",
										"value": "127.0.0.1"
									},
									{
										"key": "Client-Platform-Type",
										"value": "WEB"
									},
									{
										"key": "Client-Device-Id",
										"value": "127.0.0.1"
									},
									{
										"key": "Client-User-Id",
										"value": "{{mobileNo}}"
									},
									{
										"key": "Client-User-Agent",
										"value": "firefox5.0"
									},
									{
										"key": "Cookie",
										"value": "cookiesession1=678A8C34D3B6F498C549C66E39390D07",
										"type": "text"
									}
								],
								"body": {
									"mode": "urlencoded",
									"urlencoded": [
										{
											"key": "client_id",
											"value": "hasin",
											"type": "text"
										},
										{
											"key": "client_secret",
											"value": "LRbE9A_W",
											"type": "text"
										},
										{
											"key": "grant_type",
											"value": "client_credentials",
											"type": "text"
										}
									]
								},
								"url": {
									"raw": "{{baseUrl}}/oauth/token",
									"host": [
										"{{baseUrl}}"
									],
									"path": [
										"oauth",
										"token"
									]
								}
							},
							"status": "OK",
							"code": 200,
							"_postman_previewlanguage": "json",
							"header": [
								{
									"key": "Content-Type",
									"value": "application/json; charset=utf-8"
								},
								{
									"key": "Server",
									"value": "Microsoft-IIS/10.0"
								},
								{
									"key": "Strict-Transport-Security",
									"value": "max-age=2592000"
								},
								{
									"key": "X-Powered-By",
									"value": "ASP.NET"
								},
								{
									"key": "Date",
									"value": "Mon, 04 Mar 2024 12:45:03 GMT"
								},
								{
									"key": "content-length",
									"value": "113"
								}
							],
							"cookie": [],
							"body": "{\n    \"access_token\": \"f1b2c24f-d141-42e2-84b6-d369fe694414\",\n    \"token_type\": \"bearer\",\n    \"scope\": \"all\",\n    \"expires_in\": 19329810\n}"
						}
					]
				},
				{
					"name": "create payman",
					"event": [
						{
							"listen": "prerequest",
							"script": {
								"exec": [
									"var uuid = require('uuid')",
									"",
									"const currentDate = new Date();",
									"pm.collectionVariables.set(\"startDate\",currentDate.toISOString())",
									"const expirationDate = new Date(currentDate.setMonth(currentDate.getMonth()+6));",
									"pm.collectionVariables.set(\"expirationDate\",expirationDate.toISOString())",
									"const traceID = uuid.v4()",
									"pm.collectionVariables.set(\"traceId\",traceID)"
								],
								"type": "text/javascript"
							}
						}
					],
					"request": {
						"method": "POST",
						"header": [
							{
								"key": "app-key",
								"value": "{{AppKey}}"
							},
							{
								"key": "Content-Type",
								"value": "application/json"
							},
							{
								"key": "Accept",
								"value": "application/json"
							},
							{
								"key": "Client-Ip-Address",
								"value": "127.0.0.1"
							},
							{
								"key": "Client-Platform-Type",
								"value": "web"
							},
							{
								"key": "Client-Device-Id",
								"value": "192.168.1.1"
							},
							{
								"key": "Client-User-Id",
								"value": "{{mobileNo}}"
							},
							{
								"key": "Client-User-Agent",
								"value": "firefox5.0"
							},
							{
								"key": "mobile-no",
								"value": "{{mobileNo}}"
							},
							{
								"key": "national-code",
								"value": "{{nationalCode}}"
							},
							{
								"key": "Authorization",
								"value": "{{Authorization}}"
							},
							{
								"key": "Cookie",
								"value": "cookiesession1=678A8C34D3B6F498C549C66E39390D07"
							}
						],
						"body": {
							"mode": "raw",
							"raw": "{\n    \"payman\":{\n        \"bank_code\":\"SINAIR\",\n        \"user_id\":\"{{clientId}}\",\n        \"permission_ids\":[1,2],\n        \"contract\":{\n            \"expiration_date\":\"{{expirationDate}}\",\n            \"max_daily_transaction_count\":50,\n            \"max_monthly_transaction_count\":100,\n            \"max_transaction_amount\":500000,\n            \"start_date\":\"{{startDate}}\"\n        }\n    },\n    \"redirect_url\":\"https://localhost/\",\n    \"trace_id\":\"{{traceId}}\"\n}",
							"options": {
								"raw": {
									"language": "json"
								}
							}
						},
						"url": {
							"raw": "{{baseUrl}}/v1/payman/create",
							"host": [
								"{{baseUrl}}"
							],
							"path": [
								"v1",
								"payman",
								"create"
							]
						}
					},
					"response": [
						{
							"name": "create payman",
							"originalRequest": {
								"method": "POST",
								"header": [
									{
										"key": "app-key",
										"value": "{{AppKey}}"
									},
									{
										"key": "Content-Type",
										"value": "application/json"
									},
									{
										"key": "Accept",
										"value": "application/json"
									},
									{
										"key": "Client-Ip-Address",
										"value": "127.0.0.1"
									},
									{
										"key": "Client-Platform-Type",
										"value": "web"
									},
									{
										"key": "Client-Device-Id",
										"value": "192.168.1.1"
									},
									{
										"key": "Client-User-Id",
										"value": "{{mobileNo}}"
									},
									{
										"key": "Client-User-Agent",
										"value": "firefox5.0"
									},
									{
										"key": "mobile-no",
										"value": "{{mobileNo}}"
									},
									{
										"key": "national-code",
										"value": "{{nationalCode}}"
									},
									{
										"key": "Authorization",
										"value": "{{Authorization}}"
									},
									{
										"key": "Cookie",
										"value": "cookiesession1=678A8C34D3B6F498C549C66E39390D07"
									}
								],
								"body": {
									"mode": "raw",
									"raw": "{\n    \"payman\":{\n        \"bank_code\":\"SINAIR\",\n        \"user_id\":\"hasin\",\n        \"permission_ids\":[1],\n        \"contract\":{\n            \"expiration_date\":\"{{expirationDate}}\",\n            \"max_daily_transaction_count\":50,\n            \"max_monthly_transaction_count\":100,\n            \"max_transaction_amount\":500000,\n            \"start_date\":\"{{startDate}}\"\n        }\n    },\n    \"redirect_url\":\"http://localhost:8080/\",\n    \"trace_id\":\"{{traceId}}\"\n}",
									"options": {
										"raw": {
											"language": "json"
										}
									}
								},
								"url": {
									"raw": "{{baseUrl}}/v1/payman/create",
									"host": [
										"{{baseUrl}}"
									],
									"path": [
										"v1",
										"payman",
										"create"
									]
								}
							},
							"status": "Found",
							"code": 302,
							"_postman_previewlanguage": "plain",
							"header": [
								{
									"key": "Location",
									"value": "https://oauth.sandbox.faraboom.co/oauth/authorize?bank_id=SINAIR&boom_token=PYdqygOQ5hjGL5zrmvz2Z27mE9oPOHB2BSgt29pdVKlH3e1irADYk0WVGospspLP5EJRUkAPhVSW55wzSlAimfgYK4&redirect_uri=https%3a%2f%2fpayman2.sandbox.faraboom.co%2ffa%2foauth%2freturn&response_type=code&state=R7J5dWNglFzepXD-ULhoIA~~&client_id=12594&device_id=192.168.1.1&mobile=09352673656&amount=500000&national_code=0082205388&contract=Xtpn9QdhtGdm/5WLHfJISgbnnG3Xvs7NLQL74GfS5gkzaIJQo2xLbKjhDrI8ujJelNszrHsgrvRmOMUSssqPZxkFh2+d68VxZF0BwF/m3UR0c+j8co0GAc59dQfTNLZELmAMavo9hPIdk6GWVtn6+dt1P72fYwmPu4HJu8c/TDm69tMBMqybgaixZPT1W3LDVQ3dw+K+PYWAaAldKpoTJyU9uFmt6uq+G/0rg4BaX8p2IsSkzXA2jPS2hu/ep9OlTuf4jc4eJNq7va0TXZ3I+D4uRVKCfCVNLI7RkeoN98ZtbIVo5JMTNqxZlULkd6Pll6lyiizvMGRMGcL4DgzvAw=="
								},
								{
									"key": "Server",
									"value": "Microsoft-IIS/10.0"
								},
								{
									"key": "Strict-Transport-Security",
									"value": "max-age=2592000"
								},
								{
									"key": "X-Powered-By",
									"value": "ASP.NET"
								},
								{
									"key": "Date",
									"value": "Mon, 04 Mar 2024 12:55:08 GMT"
								},
								{
									"key": "content-length",
									"value": "0"
								}
							],
							"cookie": [],
							"body": null
						}
					]
				},
				{
					"name": "get id and activate",
					"event": [
						{
							"listen": "test",
							"script": {
								"exec": [
									"const responseJson = pm.response.json()",
									"pm.collectionVariables.set(\"paymanId\",responseJson.payman_id)"
								],
								"type": "text/javascript"
							}
						}
					],
					"request": {
						"method": "GET",
						"header": [
							{
								"key": "Authorization",
								"value": "{{Authorization}}"
							},
							{
								"key": "App-Key",
								"value": "{{AppKey}}"
							},
							{
								"key": "Content-Type",
								"value": "application/json"
							},
							{
								"key": "Accept",
								"value": "application/json"
							},
							{
								"key": "Client-Ip-Address",
								"value": "127.0.0.1"
							},
							{
								"key": "Client-Platform-Type",
								"value": "web"
							},
							{
								"key": "Client-Device-Id",
								"value": "127.0.0.1"
							},
							{
								"key": "Client-User-Id",
								"value": "{{mobileNo}}"
							},
							{
								"key": "Client-User-Agent",
								"value": "firefox5.0"
							},
							{
								"key": "Cookie",
								"value": "cookiesession1=678A8C34D3B6F498C549C66E39390D07"
							},
							{
								"key": "Accept-Language",
								"value": "fa",
								"type": "text"
							},
							{
								"key": "Device-Id",
								"value": "127.0.0.1",
								"type": "text"
							}
						],
						"url": {
							"raw": "{{baseUrl}}/v1/payman/getId?payman_code=mt1j6y4pNfqR",
							"host": [
								"{{baseUrl}}"
							],
							"path": [
								"v1",
								"payman",
								"getId"
							],
							"query": [
								{
									"key": "payman_code",
									"value": "mt1j6y4pNfqR"
								}
							]
						}
					},
					"response": [
						{
							"name": "get code and activate",
							"originalRequest": {
								"method": "GET",
								"header": [
									{
										"key": "Authorization",
										"value": "{{Authorization}}"
									},
									{
										"key": "App-Key",
										"value": "{{AppKey}}"
									},
									{
										"key": "Content-Type",
										"value": "application/json"
									},
									{
										"key": "Accept",
										"value": "application/json"
									},
									{
										"key": "Client-Ip-Address",
										"value": "127.0.0.1"
									},
									{
										"key": "Client-Platform-Type",
										"value": "web"
									},
									{
										"key": "Client-Device-Id",
										"value": "127.0.0.1"
									},
									{
										"key": "Client-User-Id",
										"value": "{{mobileNo}}"
									},
									{
										"key": "Client-User-Agent",
										"value": "firefox5.0"
									},
									{
										"key": "Cookie",
										"value": "cookiesession1=678A8C34D3B6F498C549C66E39390D07"
									},
									{
										"key": "Accept-Language",
										"value": "fa",
										"type": "text"
									},
									{
										"key": "Device-Id",
										"value": "127.0.0.1",
										"type": "text"
									}
								],
								"url": {
									"raw": "{{baseUrl}}/v1/payman/getId?payman_code=CD6j7hOdsY0u",
									"host": [
										"{{baseUrl}}"
									],
									"path": [
										"v1",
										"payman",
										"getId"
									],
									"query": [
										{
											"key": "payman_code",
											"value": "CD6j7hOdsY0u"
										}
									]
								}
							},
							"status": "OK",
							"code": 200,
							"_postman_previewlanguage": "json",
							"header": [
								{
									"key": "Content-Type",
									"value": "application/json; charset=utf-8"
								},
								{
									"key": "Server",
									"value": "Microsoft-IIS/10.0"
								},
								{
									"key": "Strict-Transport-Security",
									"value": "max-age=2592000"
								},
								{
									"key": "X-Powered-By",
									"value": "ASP.NET"
								},
								{
									"key": "Date",
									"value": "Mon, 04 Mar 2024 12:56:07 GMT"
								},
								{
									"key": "content-length",
									"value": "83"
								}
							],
							"cookie": [],
							"body": "{\n    \"payman_id\": \"0qyATpnqXeGL\",\n    \"status\": \"ACTIVE\",\n    \"deposit_number\": \"119-813-2295556-1\"\n}"
						}
					]
				},
				{
					"name": "trace contract",
					"request": {
						"method": "GET",
						"header": [
							{
								"key": "Authorization",
								"value": "{{Authorization}}"
							},
							{
								"key": "App-Key",
								"value": "{{AppKey}}"
							},
							{
								"key": "Content-Type",
								"value": "application/json"
							},
							{
								"key": "Accept",
								"value": "application/json"
							},
							{
								"key": "Client-Ip-Address",
								"value": "127.0.0.1"
							},
							{
								"key": "Client-Platform-Type",
								"value": "WEB"
							},
							{
								"key": "Client-Device-Id",
								"value": "127.0.0.1"
							},
							{
								"key": "Client-User-Id",
								"value": "{{mobileNo}}"
							},
							{
								"key": "Client-User-Agent",
								"value": "firefox5.0"
							},
							{
								"key": "Cookie",
								"value": "cookiesession1=678A8C34D3B6F498C549C66E39390D07"
							}
						],
						"url": {
							"raw": "{{baseUrl}}/v1/payman/create/trace?trace-id={{traceId}}",
							"host": [
								"{{baseUrl}}"
							],
							"path": [
								"v1",
								"payman",
								"create",
								"trace"
							],
							"query": [
								{
									"key": "trace-id",
									"value": "{{traceId}}"
								}
							]
						}
					},
					"response": [
						{
							"name": "trace contract",
							"originalRequest": {
								"method": "GET",
								"header": [
									{
										"key": "Authorization",
										"value": "{{Authorization}}"
									},
									{
										"key": "App-Key",
										"value": "{{AppKey}}"
									},
									{
										"key": "Content-Type",
										"value": "application/json"
									},
									{
										"key": "Accept",
										"value": "application/json"
									},
									{
										"key": "Client-Ip-Address",
										"value": "127.0.0.1"
									},
									{
										"key": "Client-Platform-Type",
										"value": "WEB"
									},
									{
										"key": "Client-Device-Id",
										"value": "127.0.0.1"
									},
									{
										"key": "Client-User-Id",
										"value": "{{mobileNo}}"
									},
									{
										"key": "Client-User-Agent",
										"value": "firefox5.0"
									},
									{
										"key": "Cookie",
										"value": "cookiesession1=678A8C34D3B6F498C549C66E39390D07"
									}
								],
								"url": {
									"raw": "{{baseUrl}}/v1/payman/create/trace?trace-id={{traceId}}",
									"host": [
										"{{baseUrl}}"
									],
									"path": [
										"v1",
										"payman",
										"create",
										"trace"
									],
									"query": [
										{
											"key": "trace-id",
											"value": "{{traceId}}"
										}
									]
								}
							},
							"status": "OK",
							"code": 200,
							"_postman_previewlanguage": "json",
							"header": [
								{
									"key": "Content-Type",
									"value": "application/json; charset=utf-8"
								},
								{
									"key": "Server",
									"value": "Microsoft-IIS/10.0"
								},
								{
									"key": "Strict-Transport-Security",
									"value": "max-age=2592000"
								},
								{
									"key": "X-Powered-By",
									"value": "ASP.NET"
								},
								{
									"key": "Date",
									"value": "Mon, 04 Mar 2024 12:58:03 GMT"
								},
								{
									"key": "content-length",
									"value": "393"
								}
							],
							"cookie": [],
							"body": "{\n    \"bank_code\": \"SINAIR\",\n    \"user_id\": \"hasin\",\n    \"payman_id\": \"0qyATpnqXeGL\",\n    \"status\": \"ACTIVE\",\n    \"internal_status\": \"قرارداد پیمان فعال شده\",\n    \"permission_ids\": [\n        1\n    ],\n    \"contract\": {\n        \"expiration_date\": 1725441908000,\n        \"start_date\": 1709544308000,\n        \"max_daily_transaction_count\": 50,\n        \"max_monthly_transaction_count\": 100,\n        \"max_transaction_amount\": 500000,\n        \"currency\": \"IRR\"\n    },\n    \"over_draft\": {\n        \"status\": \"DEACTIVE\"\n    }\n}"
						}
					]
				},
				{
					"name": "pay",
					"event": [
						{
							"listen": "prerequest",
							"script": {
								"exec": [
									"var uuid = require('uuid')",
									"const traceID = uuid.v4()",
									"pm.collectionVariables.set(\"paymentTraceId\",traceID)"
								],
								"type": "text/javascript"
							}
						}
					],
					"request": {
						"method": "POST",
						"header": [
							{
								"key": "Authorization",
								"value": "{{Authorization}}"
							},
							{
								"key": "App-Key",
								"value": "{{AppKey}}"
							},
							{
								"key": "Content-Type",
								"value": "application/json"
							},
							{
								"key": "Accept",
								"value": "application/json"
							},
							{
								"key": "Client-Ip-Address",
								"value": "127.0.0.1"
							},
							{
								"key": "Client-Platform-Type",
								"value": "WEB"
							},
							{
								"key": "Client-Device-Id",
								"value": "127.0.0.1"
							},
							{
								"key": "Client-User-Id",
								"value": "{{mobileNo}}"
							},
							{
								"key": "Client-User-Agent",
								"value": "firefox5.0"
							},
							{
								"key": "Cookie",
								"value": "cookiesession1=678A8C34D3B6F498C549C66E39390D07"
							}
						],
						"body": {
							"mode": "raw",
							"raw": "{\n    \"trace_id\":\"{{paymentTraceId}}\",\n    \"payman_id\":\"{{paymanId}}\",\n    \"amount\":10000,\n    \"Description\":\"transfer for user\"\n//     \"client_transaction_date\":\"2022-12-30T.01:28:09.5662061+04:30\"\n}",
							"options": {
								"raw": {
									"language": "json"
								}
							}
						},
						"url": {
							"raw": "{{baseUrl}}/v1/payman/pay",
							"host": [
								"{{baseUrl}}"
							],
							"path": [
								"v1",
								"payman",
								"pay"
							]
						}
					},
					"response": [
						{
							"name": "pay",
							"originalRequest": {
								"method": "POST",
								"header": [
									{
										"key": "Authorization",
										"value": "{{Authorization}}"
									},
									{
										"key": "App-Key",
										"value": "{{AppKey}}"
									},
									{
										"key": "Content-Type",
										"value": "application/json"
									},
									{
										"key": "Accept",
										"value": "application/json"
									},
									{
										"key": "Client-Ip-Address",
										"value": "127.0.0.1"
									},
									{
										"key": "Client-Platform-Type",
										"value": "WEB"
									},
									{
										"key": "Client-Device-Id",
										"value": "127.0.0.1"
									},
									{
										"key": "Client-User-Id",
										"value": "{{mobileNo}}"
									},
									{
										"key": "Client-User-Agent",
										"value": "firefox5.0"
									},
									{
										"key": "Cookie",
										"value": "cookiesession1=678A8C34D3B6F498C549C66E39390D07"
									}
								],
								"body": {
									"mode": "raw",
									"raw": "{\n    \"trace_id\":\"{{paymentTraceId}}\",\n    \"payman_id\":\"{{paymanId}}\",\n    \"amount\":10000,\n    \"Description\":\"transfer for user\"\n/*     \"client_transaction_date\":\"2022-12-30T.01:28:09.5662061+04:30\"\n */}",
									"options": {
										"raw": {
											"language": "json"
										}
									}
								},
								"url": {
									"raw": "{{baseUrl}}/v1/payman/pay",
									"host": [
										"{{baseUrl}}"
									],
									"path": [
										"v1",
										"payman",
										"pay"
									]
								}
							},
							"status": "OK",
							"code": 200,
							"_postman_previewlanguage": "json",
							"header": [
								{
									"key": "Content-Type",
									"value": "application/json; charset=utf-8"
								},
								{
									"key": "Server",
									"value": "Microsoft-IIS/10.0"
								},
								{
									"key": "Strict-Transport-Security",
									"value": "max-age=2592000"
								},
								{
									"key": "X-Powered-By",
									"value": "ASP.NET"
								},
								{
									"key": "Date",
									"value": "Mon, 04 Mar 2024 13:22:11 GMT"
								},
								{
									"key": "content-length",
									"value": "434"
								}
							],
							"cookie": [],
							"body": "{\n    \"reference_id\": \"00001709558528707170\",\n    \"trace_id\": \"ca3fe113-82f9-4b63-b58c-1dce5c7e6bbd\",\n    \"transaction_amount\": 10000,\n    \"transaction_time\": 1709558531250,\n    \"batch_id\": 841906,\n    \"commission_amount\": 0,\n    \"status\": \"SUCCEED\",\n    \"details\": [\n        {\n            \"reference_id\": \"00001709558528707170\",\n            \"trace_id\": \"ca3fe113-82f9-4b63-b58c-1dce5c7e6bbd_1\",\n            \"amount\": 10000,\n            \"transaction_time\": 1709545931221,\n            \"transaction_detail_type\": \"MAIN\",\n            \"status\": \"SUCCEED\"\n        }\n    ],\n    \"is_over_draft\": false\n}"
						}
					]
				},
				{
					"name": "trace payment",
					"request": {
						"method": "GET",
						"header": [
							{
								"key": "Authorization",
								"value": "{{Authorization}}"
							},
							{
								"key": "App-Key",
								"value": "{{AppKey}}"
							},
							{
								"key": "Content-Type",
								"value": "application/json"
							},
							{
								"key": "Accept",
								"value": "application/json"
							},
							{
								"key": "Client-Ip-Address",
								"value": "127.0.0.1"
							},
							{
								"key": "Client-Platform-Type",
								"value": "WEB"
							},
							{
								"key": "Client-Device-Id",
								"value": "127.0.0.1"
							},
							{
								"key": "Client-User-Id",
								"value": "{{mobileNo}}"
							},
							{
								"key": "Client-User-Agent",
								"value": "firefox 5.0"
							},
							{
								"key": "Cookie",
								"value": "cookiesession1=678A8C34D3B6F498C549C66E39390D07"
							}
						],
						"url": {
							"raw": "{{baseUrl}}/v1/payman/pay/trace?trace-id={{paymentTraceId}}&date=2024-03-04",
							"host": [
								"{{baseUrl}}"
							],
							"path": [
								"v1",
								"payman",
								"pay",
								"trace"
							],
							"query": [
								{
									"key": "trace-id",
									"value": "{{paymentTraceId}}"
								},
								{
									"key": "date",
									"value": "2024-03-04"
								}
							]
						}
					},
					"response": [
						{
							"name": "trace payment",
							"originalRequest": {
								"method": "GET",
								"header": [
									{
										"key": "Authorization",
										"value": "{{Authorization}}"
									},
									{
										"key": "App-Key",
										"value": "{{AppKey}}"
									},
									{
										"key": "Content-Type",
										"value": "application/json"
									},
									{
										"key": "Accept",
										"value": "application/json"
									},
									{
										"key": "Client-Ip-Address",
										"value": "127.0.0.1"
									},
									{
										"key": "Client-Platform-Type",
										"value": "WEB"
									},
									{
										"key": "Client-Device-Id",
										"value": "127.0.0.1"
									},
									{
										"key": "Client-User-Id",
										"value": "{{mobileNo}}"
									},
									{
										"key": "Client-User-Agent",
										"value": "firefox 5.0"
									},
									{
										"key": "Cookie",
										"value": "cookiesession1=678A8C34D3B6F498C549C66E39390D07"
									}
								],
								"url": {
									"raw": "{{baseUrl}}/v1/payman/pay/trace?trace-id={{paymentTraceId}}&date=2024-03-04",
									"host": [
										"{{baseUrl}}"
									],
									"path": [
										"v1",
										"payman",
										"pay",
										"trace"
									],
									"query": [
										{
											"key": "trace-id",
											"value": "{{paymentTraceId}}"
										},
										{
											"key": "date",
											"value": "2024-03-04"
										}
									]
								}
							},
							"status": "OK",
							"code": 200,
							"_postman_previewlanguage": "json",
							"header": [
								{
									"key": "Content-Type",
									"value": "application/json; charset=utf-8"
								},
								{
									"key": "Server",
									"value": "Microsoft-IIS/10.0"
								},
								{
									"key": "Strict-Transport-Security",
									"value": "max-age=2592000"
								},
								{
									"key": "X-Powered-By",
									"value": "ASP.NET"
								},
								{
									"key": "Date",
									"value": "Mon, 04 Mar 2024 13:25:05 GMT"
								},
								{
									"key": "content-length",
									"value": "439"
								}
							],
							"cookie": [],
							"body": "{\n    \"currency\": \"IRR\",\n    \"description\": \"transfer for user\",\n    \"destination_bank\": \"SINAIR\",\n    \"destination_deposit\": \"361-813-2295556-1\",\n    \"source_bank\": \"SINAIR\",\n    \"source_deposit\": \"119-813-2295556-1\",\n    \"transaction_type\": \"NORMAL\",\n    \"reference_id\": \"00001709558528707170\",\n    \"trace_id\": \"ca3fe113-82f9-4b63-b58c-1dce5c7e6bbd\",\n    \"transaction_amount\": 10000,\n    \"transaction_time\": 1709558528000,\n    \"batch_id\": 841906,\n    \"commission_amount\": 0,\n    \"status\": \"SUCCEED\",\n    \"is_over_draft\": false\n}"
						}
					]
				},
				{
					"name": "change contract status",
					"request": {
						"method": "POST",
						"header": [
							{
								"key": "Authorization",
								"value": "{{Authorization}}"
							},
							{
								"key": "App-Key",
								"value": "{{AppKey}}"
							},
							{
								"key": "Content-Type",
								"value": "application/json"
							},
							{
								"key": "Accept",
								"value": "application/json"
							},
							{
								"key": "Client-Ip-Address",
								"value": "127.0.0.1"
							},
							{
								"key": "Client-Platform-Type",
								"value": "WEB"
							},
							{
								"key": "Client-Device-Id",
								"value": "127.0.0.1"
							},
							{
								"key": "Client-User-Id",
								"value": "{{mobileNo}}"
							},
							{
								"key": "Client-User-Agent",
								"value": "firefox 5.0"
							},
							{
								"key": "Cookie",
								"value": "cookiesession1=678A8C34D3B6F498C549C66E39390D 07"
							}
						],
						"body": {
							"mode": "raw",
							"raw": "{\n    \"payman_id\":\"{{paymanId}}\",\n    \"new_status\":\"CANCELLED\"\n}"
						},
						"url": {
							"raw": "{{baseUrl}}/v1/payman/status/change",
							"host": [
								"{{baseUrl}}"
							],
							"path": [
								"v1",
								"payman",
								"status",
								"change"
							]
						}
					},
					"response": [
						{
							"name": "change contract status",
							"originalRequest": {
								"method": "POST",
								"header": [
									{
										"key": "Authorization",
										"value": "{{Authorization}}"
									},
									{
										"key": "App-Key",
										"value": "{{AppKey}}"
									},
									{
										"key": "Content-Type",
										"value": "application/json"
									},
									{
										"key": "Accept",
										"value": "application/json"
									},
									{
										"key": "Client-Ip-Address",
										"value": "127.0.0.1"
									},
									{
										"key": "Client-Platform-Type",
										"value": "WEB"
									},
									{
										"key": "Client-Device-Id",
										"value": "127.0.0.1"
									},
									{
										"key": "Client-User-Id",
										"value": "{{mobileNo}}"
									},
									{
										"key": "Client-User-Agent",
										"value": "firefox 5.0"
									},
									{
										"key": "Cookie",
										"value": "cookiesession1=678A8C34D3B6F498C549C66E39390D 07"
									}
								],
								"body": {
									"mode": "raw",
									"raw": "{\n    \"payman_id\":\"{{paymanId}}\",\n    \"new_status\":\"CANCELLED\"\n}"
								},
								"url": {
									"raw": "{{baseUrl}}/v1/payman/status/change",
									"host": [
										"{{baseUrl}}"
									],
									"path": [
										"v1",
										"payman",
										"status",
										"change"
									]
								}
							},
							"status": "OK",
							"code": 200,
							"_postman_previewlanguage": "json",
							"header": [
								{
									"key": "Content-Type",
									"value": "application/json; charset=utf-8"
								},
								{
									"key": "Server",
									"value": "Microsoft-IIS/10.0"
								},
								{
									"key": "Strict-Transport-Security",
									"value": "max-age=2592000"
								},
								{
									"key": "X-Powered-By",
									"value": "ASP.NET"
								},
								{
									"key": "Date",
									"value": "Mon, 04 Mar 2024 13:04:02 GMT"
								},
								{
									"key": "content-length",
									"value": "49"
								}
							],
							"cookie": [],
							"body": "{\n    \"payman_id\": \"0qyATpnqXeGL\",\n    \"status\": \"CANCELLED\"\n}"
						}
					]
				}
			]
		},
		{
			"name": "update contract",
			"request": {
				"method": "POST",
				"header": [
					{
						"key": "Authorization",
						"value": "{{Authorization}}"
					},
					{
						"key": "app-key",
						"value": "{{AppKey}}"
					},
					{
						"key": "Content-Type",
						"value": "application/json"
					},
					{
						"key": "Accept",
						"value": "application/json"
					},
					{
						"key": "Client-Ip-Address",
						"value": "127.0.0.1"
					},
					{
						"key": "Client-Platform-Type",
						"value": "web"
					},
					{
						"key": "Client-Device-Id",
						"value": "127.0.0.1"
					},
					{
						"key": "Client-User-Id",
						"value": "{{mobileNo}}"
					},
					{
						"key": "Client-User-Agent",
						"value": "firefox 5.0"
					},
					{
						"key": "mobile-no",
						"value": "{{mobileNo}}"
					},
					{
						"key": "national-code",
						"value": "{{nationalCode}}"
					},
					{
						"key": "Cookie",
						"value": "cookiesession1=678A8C34D3B6F498C549C66E39390D\\ 07"
					},
					{
						"key": "Accept-Language",
						"value": "fa",
						"type": "text"
					},
					{
						"key": "Device-Id",
						"value": "127.0.0.1",
						"type": "text"
					}
				],
				"body": {
					"mode": "raw",
					"raw": "{\n    \"payman_id\":\"gvb45yeybe45b\",\n    \"expiration_date\":\"2022-12-30T.01:28:09.5662061+04:30\",\n    \"max_daily_transaction_count\":10,\n    \"max_monthly_transaction_count\":30,\n    \"max_transaction_amount\":200000.0,\n    \"start_date\":\"2022-11-22T.08:28:09.5662061+04:30\",\n    \"daily_max_Transaction_amount\":456000000.0,\n    \"redirect_url\":\"http://localhost/\"\n\n}",
					"options": {
						"raw": {
							"language": "json"
						}
					}
				},
				"url": {
					"raw": "{{baseUrl}}/v1/payman/update",
					"host": [
						"{{baseUrl}}"
					],
					"path": [
						"v1",
						"payman",
						"update"
					]
				}
			},
			"response": []
		},
		{
			"name": "single contract report",
			"request": {
				"method": "GET",
				"header": [
					{
						"key": "Authorization",
						"value": "{{Authorization}}"
					},
					{
						"key": "App-Key",
						"value": "{{AppKey}}"
					},
					{
						"key": "Content-Type",
						"value": "application/json"
					},
					{
						"key": "Accept",
						"value": "application/json"
					},
					{
						"key": "Client-Ip-Address",
						"value": "127.0.0.1"
					},
					{
						"key": "Client-Platform-Type",
						"value": "WEB"
					},
					{
						"key": "Client-Device-Id",
						"value": "127.0.0.1"
					},
					{
						"key": "Client-User-Id",
						"value": "{{mobileNo}}"
					},
					{
						"key": "Client-User-Agent",
						"value": "firefox5.0"
					},
					{
						"key": "Cookie",
						"value": "cookiesession1=678A8C34D3B6F498C549C66E39390D07"
					},
					{
						"key": "Accept-Language",
						"value": "fa",
						"type": "text"
					},
					{
						"key": "Device-Id",
						"value": "127.0.0.1",
						"type": "text"
					}
				],
				"url": {
					"raw": "{{baseUrl}}/v1/payman/report/{{paymanId}}",
					"host": [
						"{{baseUrl}}"
					],
					"path": [
						"v1",
						"payman",
						"report",
						"{{paymanId}}"
					]
				}
			},
			"response": []
		},
		{
			"name": "list contract transactions",
			"request": {
				"method": "POST",
				"header": [
					{
						"key": "App-Key",
						"value": "{{AppKey}}"
					},
					{
						"key": "Content-Type",
						"value": "application/json"
					},
					{
						"key": "Accept",
						"value": "application/json"
					},
					{
						"key": "Client-Ip-Address",
						"value": "127.0.0.1"
					},
					{
						"key": "Client-Platform-Type",
						"value": "WEB"
					},
					{
						"key": "Client-Device-Id",
						"value": "127.0.0.1"
					},
					{
						"key": "Client-User-Id",
						"value": "{{mobileNo}}"
					},
					{
						"key": "Client-User-Agent",
						"value": "firefox5.0"
					},
					{
						"key": "Authorization",
						"value": "{{Authorization}}"
					},
					{
						"key": "Cookie",
						"value": "cookiesession1=678A8C34D3B6F498C549C66E39390D07"
					}
				],
				"body": {
					"mode": "raw",
					"raw": "{\n\"filter\": {\n    \"payman_ids\": [\n        \"tuxxCGxFFLRl\",\n        \"QIq64G4NhrR7\",\n        \"0lzgY1UPF3I8\"\n    ],\n    \"transaction_type\": \"DIRECT_DEBIT\",\n    \"from_transaction_amount\": \"1\",\n    \"to_transaction_amount\": \"20000\",\n    \"from_transaction_date\": 1613648132000,\n    \"to_transaction_date\": 1668587752000,\n    \"payamn_statuses\": [\n        \"ACTIVE\",\n        \"CANCELLED\",\n        \"DEACTIVE\"\n    ],\n    \"transaction_statuses\": [\n        \"SUCCEED\",\n        \"FAILED\"\n    ]\n}}"
				},
				"url": {
					"raw": "{{baseUrl}}/v1/payman/transactions/",
					"host": [
						"{{baseUrl}}"
					],
					"path": [
						"v1",
						"payman",
						"transactions",
						""
					]
				}
			},
			"response": []
		},
		{
			"name": "list organization contracts",
			"request": {
				"method": "POST",
				"header": [
					{
						"key": "Authorization",
						"value": "{{Authorization}}"
					},
					{
						"key": "Content-Type",
						"value": "application/json"
					},
					{
						"key": "app-key",
						"value": "{{AppKey}}"
					},
					{
						"key": "Client-Ip-Address",
						"value": "127.0.0.1"
					},
					{
						"key": "Client-Platform-Type",
						"value": "WEB"
					},
					{
						"key": "Client-Device-Id",
						"value": "127.0.0.1"
					},
					{
						"key": "Client-User-Id",
						"value": "{{mobileNo}}"
					},
					{
						"key": "Client-User-Agent",
						"value": "firefox5.0"
					},
					{
						"key": "Cookie",
						"value": "cookiesession1=678A8C34D3B6F498C549C66E39390D 07"
					}
				],
				"body": {
					"mode": "raw",
					"raw": "{\n    \"filter\": {\n        \"bank_code\": \"SINAIR\",\n        \"expiration_date_from\": \"23-01-2022T53.911:41:11Z\",\n        \"expiration_date_to\": \"23-12-2023T53.911:41:11Z\",\n        \"max_daily_transactions_count_from\": 1,\n        \"max_daily_transactions_count_to\": 150,\n        \"max_monthly_transactions_count_from\": 1,\n        \"max_monthly_transactions_count_to\": 150,\n        \"start_date_from\": \"01-01-2022T53.911:41:11 Z\",\n        \"start_date_to\": \"23-12-2023T53.911:41:11 Z\",\n        \"statuses\": [\n            \"WAITING_FOR_CONFIRM\",\n            \"ACTIVE\"\n        ],\n        \"transaction_max_amount_from\": 1,\n        \"transaction_max_amount_to\": 20000000,\n        \"payman_id\": \"wO1rjZ8Nwecd\",\n        \"user_ids\": [\n            \"10200\",\n            \"sampleUserId 9876\"\n        ],\n        \"permission_ids\": [\n            1,\n            2\n        ]\n    },\n    \"offset\": 0,\n    \"length\": 15\n}"
				},
				"url": {
					"raw": "{{baseUrl}}/v1/payman/search",
					"host": [
						"{{baseUrl}}"
					],
					"path": [
						"v1",
						"payman",
						"search"
					]
				}
			},
			"response": []
		}
	],
	"event": [
		{
			"listen": "prerequest",
			"script": {
				"type": "text/javascript",
				"exec": [
					""
				]
			}
		},
		{
			"listen": "test",
			"script": {
				"type": "text/javascript",
				"exec": [
					""
				]
			}
		}
	],
	"variable": [
		{
			"key": "baseUrl",
			"value": "https://payman2.sandbox.faraboom.co",
			"type": "string"
		},
		{
			"key": "AppKey",
			"value": "<APP-KEY>",
			"type": "string"
		},
		{
			"key": "clientId",
			"value": "<CLIENT-ID>",
			"type": "string"
		},
		{
			"key": "clientSecret",
			"value": "<CLIENT-SECRET>",
			"type": "string"
		},
		{
			"key": "mobileNo",
			"value": "<USER-PHONE-NUMBER>",
			"type": "string"
		},
		{
			"key": "nationalCode",
			"value": "<USER-NATIONAL-ID>",
			"type": "string"
		},
		{
			"key": "Authorization",
			"value": "Bearer <AUTH-TOKEN>",
			"type": "string"
		},
		{
			"key": "paymanId",
			"value": "<PAYMAN-ID>",
			"type": "string"
		},
		{
			"key": "startDate",
			"value": "2024-03-04T13:17:14.286Z",
			"type": "string"
		},
		{
			"key": "expirationDate",
			"value": "2024-09-04T13:17:14.286Z",
			"type": "string"
		},
		{
			"key": "traceId",
			"value": "fe6552c5-925c-4877-acbb-acc97889854a",
			"type": "string"
		},
		{
			"key": "paymentTraceId",
			"value": "ca3fe113-82f9-4b63-b58c-1dce5c7e6bbd"
		}
	]
}

```