![1594966842702](C:\Users\lelec_1.TUJIA\AppData\Roaming\Typora\typora-user-images\1594966842702.png)

dao服务接口？



"/suggestion/index2"

com.tujia.tns.suggestion.web.controller.SuggestionController.suggestIndex2(@RequestBody BaseRequest<SuggestionParameter> dataRequest)



## suggest业务逻辑

http://wiki.corp.tujia.com/confluence/pages/viewpage.action?pageId=29798834

com.tujia.tns.suggestion.service.impl.SuggestionServiceImpl.suggest(SuggestionParameter parameter)

### 参数验证

![1595209927171](C:\Users\lelec_1.TUJIA\AppData\Roaming\Typora\typora-user-images\1595209927171.png)

`SuggestionParameter`

```json
"enumWrapperId":"waptujia001",     ##渠道
"EnumSourceType":1,
"CityId":23,
"CityName":"上海",
"SourceCode":"App_UnitList_Mainland",      ##分框
"DeviceId":"3FD8FDC4BFCA343322C8F81992EA38E6",
"CustomerLoginId":11934533,
"Query":"隐贤居",    ##用户输入
"Coord":"31.3110870,121.4042367",
"Version":"6.51"
```

### 入参信息修改

```java
rebuildParameter(sourceCodeEnum, parameter);
```

### 上下文信息保存

```java
SuggestionContext context = new SuggestionContext(sourceCodeEnum, parameter);
```

```java
public class SuggestionContext {
    private boolean debugEnabled;
    /**
     * 原始参数
     */
    private SuggestionParameter originPara;

    private QueryBean queryBean;
    /** 是否使用新的展示名称 */
    private boolean useNewDisplayCfg = false;
    /**
     * 本服务单独控制的分桶参数
     */
    private String bucket;
    /**
     * 上下文创建时保存访问时间
     */
    private long timestamp;
    /**
     * 前端A/B分桶参数
     */
    private Map<String, String> abTest = Maps.newHashMap();

    /**
     * 同城展示记录数量，null表示不控制数量。主要在二框搜索异地知名地标，折叠部分同城房屋门店展示
     */
    private Integer sameCityShowCount;
    /**
     * 召回、排序后的数据若只有房屋或门店等情况时，将其中的房屋id列表存储至该字段，用于用户点击直搜时传至列表页展示房屋
     */
    private List<Long> directSearchList;
    /**
     * 类似directSearchList，此处存储的是15个词条中的门店id列表
     */
    private List<Long> directHotelList;
    /**
     * 是否进入了直搜扩充房屋列表场景
     */
    private boolean directSearchScene = false;

    /**用于数据统计*/
    private DataStatisticBean dataStatis;
    /**
     * 是否为四方渠道（非小程序、PC等）
     */
    private boolean parentWrapper = false;
```



### 噪声过滤与数据预处理

```java
queryAdjustmentService.adjust(context);
context.getQueryBean().setSuggestionContext(context);
```

```java
public void adjust(SuggestionContext context) {    
    if (context == null) {        return;    }    
    for (AbstractQueryAdjustFilter filter : filters) {        			    		filter.adjust(context);    
}}
```



```
SuggestionParameter(enumWrapperId=waptujia001, rebuildWrapperId=waptujia001, enumSourceType=null, cityId=null, cityName=北京, sourceCode=Common_Blank1, deviceId=1111, query=王府井, coord=39.902915,116.421750, version=206, typeGroupCount=0, debug=true, abTest={SuggestionStyleInformationV86=A}, forCtripHotel=false, customerLoginId=, forCrn=false)
QueryBean(originPara=SuggestionParameter(enumWrapperId=waptujia001, rebuildWrapperId=waptujia001, enumSourceType=null, cityId=null, cityName=北京, sourceCode=Common_Blank1, deviceId=1111, query=王府井, coord=39.902915,116.421750, version=206, typeGroupCount=0, debug=true, abTest={SuggestionStyleInformationV86=A}, forCtripHotel=false, customerLoginId=, forCrn=false), 
adjustQry=null, adjustQryNoFilter=null, adjustQryUnext=null, adjustQryPy=null, adjustQryUnextPy=null, adjustQryNoFilterPy=null, queryWords=null, queryTermIDF=null, queryTermIDFWeight=null, queryWordsPy=null, bCity=null, qCity=null, qCityId=null, province=null, intentCity=null, qDestination=null, rmIntentDestQryInfo=null, scene=null, tags=null, featureTagList=null, qryOnlyCityTags=false, suggestionContext=null, queryStringWeight=null, positionType=one, internalOverseas=NONE, tujiaVersionOver206=true)
```

### 分词

```java
String adjustQry = context.getQueryBean().getAdjustQry();
        if (StringUtils.isNotBlank(adjustQry)) {
            List<String> words = tokenizerService.tokenize(adjustQry);
            context.getQueryBean().setQueryWords(words);
        } else {
      TMonitor.recordOne(TMonitorConstants.SUGGEST_EMPTY_RESULT_BY_NULL_INPUT);
            return SuggestionResult.EMPTY_SUGGEST_RESULT;
        }
```

![1595212253255](C:\Users\lelec_1.TUJIA\AppData\Roaming\Typora\typora-user-images\1595212253255.png)

### 意图识别

```
intentExtractService.extract(context);
```

```
DefaultExtractStrategy.extract(SuggestionContext context);
```

1. 设置城市????????

```java
queryBean.setQCity(destinationInfo.getCityName());
queryBean.setQCityId(destinationInfo.getCityId());
```

2. 用户输入为城市时且为一框逻辑时，重写query



3. 从分词结果中找乡镇名对应城市



4. 从坐标查位置城市



5. 省份设置



### 重新分词adjustQuery





### 数据召回与初步排序

```
Queue<SuggestSolrBean> convergedQueue = suggestRecallService.recallAndConverge(context);
```

返回从slor中取出的数据