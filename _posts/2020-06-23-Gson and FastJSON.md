---
layout: post
title: Gson and FastJSON
tags: Web
---
## 背景
gson与fastjson使用，fastjson常有安全漏洞但快

## GSON
### 字符串与JsonObject互转
```
JsonObject jsonObject = JsonParser.parseString(json).getAsJsonObject();
String str = jsonObject.toString();
```

### 字符串与类互转
```
User user = new Gson().fromJson(json, User.class);
String str = new Gson().toJson(user)
```

### 字符串与列表互转
```
List<User> users = new Gson().fromJson(json, new TypeToken<List<User>>(){}.getType());
String str = new Gson().toJson(users);
```

### 组装JsonObject
```
JsonObject jsonObject = new JsonObject();
jsonObject.addProperty("a", "a");
jsonObject.addProperty("b", 1);
jsonObject.add("a", new Gson().toJsonTree(new ArrayList<List<String>>()));
```

### JsonObject获得值、JsonArray或JsonObject
```
jsonObject.get("code");
JsonArray jsonArray = jsonObject.getAsJsonArray("data");
JsonObject jsonObj = jsonArray.get(0).getAsJsonObject();
```

## FastJSON
### 字符串与JSONObject互转
```
JSONObject jsonObject = JSON.parseObject(json);
JSONArray jsonArray = JSON.parseArray(json);
String str = jsonObject.toString();
```

### 字符串与类互转
```
Person person = JSONObject.parseObject(str, Person.class);
String str = JSONObject.toJSON(person).toString();
```

### 字符串与列表互转
```
List<Person> list = JSONArray.parseArray(str, Person.class);
String str = JSONArray.toJSON(person).toString();
```

### 组装JSONObject
```
JSONObject jsonObject = new JSONObject();
jsonObject.put("a", new ArrayList<List<String>>());
```

### JSONObject获得值、JSONArray或JSONObject
```
jsonObject.get("code");
JSONArray jsonArray = jsonObject.getJSONArray("data");
JSONObject jsonObj = jsonArray.getJSONObject(0);
```