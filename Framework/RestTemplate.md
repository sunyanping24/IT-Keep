# RestTemplate

由Spring Web模块提供的一个组件，可以作为一个HttpClient访问Rest服务。常用的同类型HttpClient有：JDK（`HttpURLConnection`）、Apache(`HttpClient`)、Square(`OKHttp`)等。

# 使用方法

## POST
- POST请求，发送json格式的请求数据，数据在body中
```
HttpHeaders httpHeaders = new HttpHeaders();
httpHeaders.setContentType(MediaType.APPLICATION_JSON);
httpHeaders.add("Authorization", "Bearer " + UGClientUtil.getAccessToken(ugConfig));
HttpEntity httpEntity = new HttpEntity<>(objectMapper.toJson(dataList), httpHeaders);
ResponseEntity<Object> response = restTemplate.postForEntity(segments, httpEntity, Object.class);
```