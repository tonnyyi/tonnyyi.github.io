---
title: Spring RestTemplate用法
tags:
  - spring
  - http
  - resttemplate 
categories:
  - java
  - spring
date: 2020-08-12 11:37:12
---



### Get请求

`getForEntity` 类方法返回的数据类型为 `ResponseEntity<T>` , 然后重载方法, 以不同的方式传递参数

- `getForEntity(String url, Class<T> responseType, Object... uriVariables)`

- `getForEntity(String url, Class<T> responseType, Map<String, ?> uriVariables)`

- `getForEntity(URI url, Class<T> responseType)`

示例: 

```java
// 1
ResponseEntity<Yiyan> entity1 = restTemplate
    .getForEntity("https://v1.hitokoto.cn/?c={1}&encode={2}", Yiyan.class, "h", "json");

// 2
Map<String, String> map = new HashMap<>();
map.put("1", "i");
map.put("2", "json");
ResponseEntity<Yiyan> entity2 = restTemplate
    .getForEntity("https://v1.hitokoto.cn/?c={1}&encode={2}", Yiyan.class, map);

// 3
String url3 = "https://v1.hitokoto.cn/?c={1}&encode={2}";
URI uri3 = UriComponentsBuilder.fromUriString(url3).build("c", "json");
ResponseEntity<Yiyan> entity3 = restTemplate.getForEntity(uri3, Yiyan.class);
```


如果希望接口直接返回需要的数据类型, 则可以使用下面这组方法, 返回的数据类型就是传入的`T`

- `getForObject(String url, Class<T> responseType, Object... uriVariables)`
- `getForObject(String url, Class<T> responseType, Map<String, ?> uriVariables)`
- `getForObject(URI url, Class<T> responseType)`

示例:

```java
// 1
Yiyan yiyan1 = restTemplate
    .getForObject("https://v1.hitokoto.cn/?c={1}&encode={2}", Yiyan.class, "h", "json");
System.out.println(yiyan1);

// 2
Map<String, String> map = new HashMap<>();
map.put("1", "i");
map.put("2", "json");
Yiyan yiyan2 = restTemplate.getForObject("https://v1.hitokoto.cn/?c={1}&encode={2}", Yiyan.class, map);
System.out.println(yiyan2);

// 3
String url3 = "https://v1.hitokoto.cn/?c={1}&encode={2}";
URI uri3 = UriComponentsBuilder.fromUriString(url3).build("c", "json");
Yiyan yiyan3 = restTemplate.getForObject(uri3, Yiyan.class);
```



### Post

Post和Get方法一样, 有`postForObject`和`postForEntity`方法

- `postForObject(URI url, @Nullable Object request, Class<T> responseType)`
- `postForObject(String url, @Nullable Object request, Class<T> responseType, Object... uriVariables)`
- `postForObject(String url, @Nullable Object request, Class<T> responseType, Map<String, ?> uriVariables)`

`postForEntity`类方法

- `postForEntity(URI url, @Nullable Object request, Class<T> responseType)`
- `postForEntity(String url, @Nullable Object request, Class<T> responseType, Object... uriVariables)`
- `postForEntity(String url, @Nullable Object request, Class<T> responseType, Map<String, ?> uriVariables)`

除此之外, 还有`postForLocation`, 但是这类方法的返回值为`URI`, 没法接收响应体



### exchange

这组方法偏底层, 比如需要设置请求头时使用

- `exchange(RequestEntity<?> requestEntity, Class<T> responseType)`
- `exchange(URI url, HttpMethod method, @Nullable HttpEntity<?> requestEntity, Class<T> responseType)`
- `exchange(String url, HttpMethod method, @Nullable HttpEntity<?> requestEntity, Class<T> responseType, Object... uriVariables)`
- `exchange(String url, HttpMethod method, @Nullable HttpEntity<?> requestEntity, Class<T> responseType, Map<String, ?> uriVariables)`

参数示例:

```java
// URI构造方式
// 1
URI uri = new URI("https://v1.hitokoto.cn/?c=h&encode=json");
// 2
URI uri = new UriTemplate("https://v1.hitokoto.cn/?c={1}&encode={2}").expand("h", "json");
// 或者 
Map<String, Object> map = new HashMap<>();
map.put("1", "h");
map.put("2", "json");
URI uri2 = new UriTemplate(url).expand(map);
// 3
URI uri = UriComponentsBuilder.fromUriString("https://v1.hitokoto.cn/?c={1}&encode={2}").build("h", "json");

// HttpEntity构造方式
HttpHeaders httpHeaders = new HttpHeaders();
HttpEntity<User> httpEntity = new HttpEntity<>(objBody, httpHeaders);
// RequestEntity继承了HttpEntity
RequestEntity<User> entity1 = RequestEntity.post(uri).header("Authorization", "Bearer xxxx").body(user);
```

请求示例:

```java
// 1
String url = "https://v1.hitokoto.cn/?c={1}&encode={2}";
URI uri = UriComponentsBuilder.fromUriString(url).build("h", "json");
RequestEntity<Void> request = RequestEntity.get(uri).accept(MediaType.APPLICATION_JSON)
    .header("Authorization", "123").build();
ResponseEntity<Yiyan> entity4 = restTemplate.exchange(request, Yiyan.class);

// 2
URI uri = new URI("https://gorest.co.in/public-api/users");

HttpHeaders httpHeaders = new HttpHeaders();
httpHeaders.add("Authorization", "Bearer XXXXXX");
httpHeaders.setContentType(MediaType.APPLICATION_JSON_UTF8);

User user = new User();
user.setFirst_name("Brian");
user.setLast_name("Ratke");
user.setGender("male");
user.setEmail("lew19@163.com");
user.setStatu("active");
HttpEntity<User> httpEntity = new HttpEntity<>(user, httpHeaders);

ResponseEntity<User> entity = restTemplate.exchange(uri, HttpMethod.POST, httpEntity, User.class);
System.out.println(entity);

// 3
ResponseEntity<Yiyan> entity2 = restTemplate.exchange("https://v1.hitokoto.cn/?c={1}&encode={2}", HttpMethod.GET, null, Yiyan.class, "h", "json");

// 4 略
```



下面这类方法是为了支持泛型, 比如, 返回数据类型是List, 但用上面方法无法进一步设置List内元素类型

- `exchange(RequestEntity<?> requestEntity, ParameterizedTypeReference<T> responseType)`
- `exchange(URI url, HttpMethod method, @Nullable HttpEntity<?> requestEntity, ParameterizedTypeReference<T> responseType)`
- `exchange(String url, HttpMethod method, @Nullable HttpEntity<?> requestEntity, ParameterizedTypeReference<T> responseType, Object... uriVariables)`
- `exchange(String url, HttpMethod method, @Nullable HttpEntity<?> requestEntity, ParameterizedTypeReference<T> responseType, Map<String, ?> uriVariables)`

