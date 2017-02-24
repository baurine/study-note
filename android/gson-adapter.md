# 用 Gson TypeAdapter 解析非常规 Json

在 Android 开发中，我们一般用 Retrofit + Gson 来请求网络 API 并解析返回的 Json，一般来说，我们给 Retrofit 配置一个默认的 Gson 实例就行了，如下所示：

```java
Retrofit retrofit = new Retrofit.Builder()
        .baseUrl(BASE_URL)
        .addConverterFactory(GsonConverterFactory.create())
        .build();
```

但有时候我们会遇到一些非常规的 Json，比如以下两种情况：

1. 获取一个 Feed 流，Feed 流里有不同种类的数据，以微信朋友圈为例，有纯文本，有文本和图片的混合，还有分享的链接。
1. 返回的 Json 中相同对象的 key 是变化的，而不是固定的，比如用 id 或 日期 作为 key (后面会举例)。这种 Json 对于动态语言比如 JavaScript 是比较友好的，但对于静态语言来说就是灾难。

遇到这两种情况，默认的 Gson 实例就不够用了，我们必须实现自定义的 Gson TypeAdapter 来解析某些特定类型，然后把这个 adapter 注册给 Gson 实例，告诉它，以后遇到这种类型的数据就教给我来处理，你就别管了。

## 1. 数组中包含多种类型

我们先来看第一种情况，就以 Feed 流为例，一般情况下我们得到的是一个 Json 数组，数组中包含多种类型的对象。我们会和服务端约定，这些对象，至少会有一个相同的属性，比如 type，由这个属性来标志这个对象的类别。所以在解析时，我们先取出这个 type 属性的值，根据它的值来解析整个对象。

假设我们得到的 Json 是这样的：

```json
[
    {
        "type": "text",
        "text": "work hard, enjoy life"
    },
    {
        "type": "image",
        "desc": "beautiful!",
        "url" : "https://unsplash.it/200/200?random"
    },
    {
        "type": "link",
        "comment": "my github",
        "link": "https://github.com/baurine"
    }
]
```

这三种 model 假设分别为 TextModel, ImageModel, LinkModel，那么它们应该有一个共同的基类 BaseModel:

```java
public class BaseModel {
    public String type;
}

public class TextModel extends BaseModel {
    public String text;
}

public class ImageModel extends BaseModel {
    public String desc;
    public String url;
}

public class LinkModel extends BaseModel {
    public String comment;
    public String link;
}
```

我们定义 getFeeds API，注意返回值的 List 中的元素必须是 BaseModel 类型：

```java
@GET("feeds")
Call<List<BaseModel>> getFeeds(@Query("page") int page);
```

然后实现我们自定义的 Json 解析器 BaseModelAdapter：

```java
public static class BaesModelAdapter implements JsonDeserializer<BaseModel> {
    @Override
    public BaseModel deserialize(JsonElement json, Type typeOfT,
                                JsonDeserializationContext context)
            throws JsonParseException {

        JsonObject jsonObj = json.getAsJsonObject();
        String type = jsonObj.get("type").getAsString();

        if (type.equals("text")) {
            return new Gson().fromJson(json, TextModel.class);
        } else if (type.equals("image")) {
            return new Gson().fromJson(json, ImageModel.class);
        } else if (type.equals("link")) {
            return new Gson().fromJson(json, LinkModel.class);
        }
        return null;
    }
}
```

把这个解析器注册到 Gson 实例中，告诉它用这个解析器来解析 BaseModel 类型的数据：

```java
Gson gson = new GsonBuilder()
        .setDateFormat("yyyy'-'MM'-'dd'T'HH':'mm':'ss'.'SSS'Z'")
        .registerTypeAdapter(BaseModel.class, new BaseModelAdapter())
        .create();
```

用新的 Gson 实例替代默认的 Gson 实例：

```java
Retrofit retrofit = new Retrofit.Builder()
        .baseUrl(BASE_URL)
        .addConverterFactory(GsonConverterFactory.create(gson))
        .build();
```

## 2. 动态 Key

比如有一个获取城市列表的 API，我们期待的 Json 是这样的：

```json
{
    "cities": [
        {
            "id": 1,
            "name": "Beijing"
        },
        {
            "id": 2,
            "name": "Shanghai"
        }
    ]
}
```

结果得到的 Json 是这样的：

```json
{
    "cities": {
        "1": {
            "name": "Beijing"
        },
        "2": {
            "name": "Shanghai"
        }
    }
}
```

又比如有一个天气预报的 API，我们期待的 Json 是这样的：

```json
{
    "forecasts": [
        {
            "date": "2017-02-08T19:00:00+08:00",
            "temperature": 13
        },
        {
            "date": "2017-02-09T19:00:00+08:00",
            "temperature": 14
        },
    ]
}
```

结果我们得到的 Json 是这样的：

```json
{
    "forecasts": {
        "2017-02-08T19:00:00+08:00": {
            "temperature": 13
        },
        "2017-02-09T19:00:00+08:00": {
            "temperature": 14
        }
    }
}
```

由于历史原因，你可能无法要求服务器修改返回结构。所以我们必须自己在客户端手动转换，把实际得到的 Json 转成期待的 Json。

以天气预报的 API 为例，首先，我们来定义 Model，我们要把 forecasts 从对象转换成 List：

```java
public class Weather {
    public Forecasts forecasts;

    //////////////////////////
    public static class Forecasts {
        public List<Forecast> forecastList;

        public Forecasts(List<Forecast> forecastList) {
            this.forecastList = forecastList;
        }
    }

    public static class Forecast {
        public Date date;
        public int temperature;
    }
}
```

因为我们要手动把 forecasts 从对象转换成 List，所以必须定义它的 Gson TypeAdapter。

我们来思考一下，怎么才能把上面实际得到的 Json 转换成期待的 Json 呢，很简单，遍历 forecasts 的每一个子对象，把作为 key 的日期值，放到它自身的对象中，增加一个 date 字段来存储它的值。

我们按这种思路来实现这个 adapter：

```java
public static class ForecastsAdapter
        implements JsonDeserializer<Forecasts> {
    @Override
    public Forecasts deserialize(
            JsonElement json,
            Type typeOfT,
            JsonDeserializationContext context)
            throws JsonParseException {
        Gson gson = new Gson();
        List<Forecast> forecastList = new ArrayList<>();
        Set<Map.Entry<String, JsonElement>> entries = json.getAsJsonObject().entrySet();
        for (Map.Entry<String, JsonElement> entry : entries) {
            String key = entry.getKey();
            JsonElement value = entry.getValue();
            value.getAsJsonObject().addProperty("date", key);
            forecastList.add(gson.fromJson(value, Forecast.class));
        }
        return new Forecasts(forecastList);
    }
}
```

然后把这个 adapter 注册到 Gson 实例以及把新的 Gson 实例配置到 Retrofit 的步骤就和上面一样了。

什么？你说要是 key 是动态的，又有多种类型，那怎么办，Apple-Pen 呗 ("I have a pen, I have an apple ...")，把上面的代码组合起来就行了。不过遇上那样的后端代码，我只能表示呵呵了。 
