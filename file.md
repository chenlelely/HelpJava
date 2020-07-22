<<<<<<< HEAD
![1594966842702](C:\Users\lelec_1.TUJIA\AppData\Roaming\Typora\typora-user-images\1594966842702.png)

dao服务接口？



"/suggestion/index2"

com.tujia.tns.suggestion.web.controller.SuggestionController.suggestIndex2(@RequestBody BaseRequest<SuggestionParameter> dataRequest)



## suggest业务逻辑

http://wiki.corp.tujia.com/confluence/pages/viewpage.action?pageId=29798834

com.tujia.tns.suggestion.service.impl.SuggestionServiceImpl.suggest(SuggestionParameter parameter)

### 参数验证

![1595209927171](C:\Users\lelec_1.TUJIA\AppData\Roaming\Typora\typora-user-images\1595209927171.png)

[SuggestionNew接口文档](http://wiki.corp.tujia.com/confluence/pages/viewpage.action?pageId=29800441)：

#### 入参SuggestionParameter

```json
"enumWrapperId":"waptujia001", ##渠道，存储入参的wrapperId，区分最初请求渠道的功能，如判断是否进入6月版
"EnumSourceType":1,
"CityId":23,
"CityName":"上海",
"SourceCode":"App_UnitList_Mainland",##应用类型编码，如："分框" = "Common_Blank2_Mainland"
"DeviceId":"3FD8FDC4BFCA343322C8F81992EA38E6",
"CustomerLoginId":11934533,
"Query":"隐贤居",    ##用户输入
"Coord":"31.3110870,121.4042367",##用户所在地区经纬度
"Version":"6.51"##调用本接口的应用版本信息

private String rebuildWrapperId;
@ApiModelProperty(value = "guesslike接口专用字段")
private int typeGroupCount;

@ApiModelProperty(value = "是否展示加权明细")
private boolean debug;

@ApiModelProperty(value = "前端A/B分桶参数", example = "")
private Map<String, String> abTest;

@ApiModelProperty("判断是否只返回已分销、接入的Ctrip数据，默认为false")
private boolean forCtripHotel = false;

@ApiModelProperty("是否支持crn，若为true，则支持6月版新样式，默认为false")
private boolean forCrn = false;
```

#### 返回参数SuggestionResult

```java
@ApiModelProperty(value = "渠道")
private String channel;
@ApiModelProperty(value = "推荐列表")
private List<SuggestionResultItem> suggestionResults;
@ApiModelProperty(value = "同城展示记录数量，null表示不控制数量。主要在二框搜索异地知名地标，折叠部分同城房屋门店展示")
private Integer sameCityShowCount;
@ApiModelProperty(value = "召回、排序后的数据有房屋或门店等情况时，存储直搜房屋id列表")
private List<Long> directSearchList;
@ApiModelProperty(value = "召回、排序后的数据有房屋或门店等情况时，存储直搜门店id列表")
private List<Long> directHotelList;
@ApiModelProperty(value = "附加随行数据,为细化分桶范围或进行数据追踪，现增加随行数据，处理更为灵活。")
private Map<String, String> additionalData;
```

- #### SuggestionResults

http://wiki.corp.tujia.com/confluence/pages/viewpage.action?pageId=29800441

```java

```



### 入参信息修改

```java
rebuildParameter(sourceCodeEnum, parameter);
```

### 上下文信息保存

```java
SourceCodeEnum sourceCodeEnum = SourceCodeEnum.findByCode(parameter.getSourceCode());
```

```java
SuggestionContext context = new SuggestionContext(sourceCodeEnum, parameter);
```

```java
public class SuggestionContext {
    private boolean debugEnabled;
    /**原始参数*/
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

1. 设置城市

```java
queryBean.setQCity(destinationInfo.getCityName());
queryBean.setQCityId(destinationInfo.getCityId());
```

2. 用户输入为城市时且为一框逻辑时，重写query



3. 从分词结果中找乡镇名对应城市



4. 从坐标查位置城市



5. 省份设置



 ![img](http://wiki.corp.tujia.com/confluence/download/attachments/29798834/%E7%89%B9%E8%89%B2%E6%8F%90%E5%8F%96.png?version=1&modificationDate=1566819979000&api=v2) 

```java
// tag
        Map<String, String> feaTagRestQMap = getFeatureTagsRestQry(context);
        if (feaTagRestQMap != null) {
            String jsonStr = feaTagRestQMap.get("tagList");
            List<FeatureTag> featureTagList = JsonUtils.readValue(jsonStr, new TypeReference<List<FeatureTag>>() {
            });
            queryBean.setFeatureTagList(featureTagList);

            String jsonNameStr = feaTagRestQMap.get("tagNameList");
            List<String> tags = JsonUtils.readValue(jsonNameStr, new TypeReference<List<String>>() {
            });
            queryBean.setTags(tags);

            // 用户输入除去特色内容
            String restQry = feaTagRestQMap.get("restQry");
            String restQryNoFilter = context.getQueryBean().getAdjustQryNoFilter();
            // 剩余内容不能为目的地
            if (StringUtils.isNotBlank(restQry) && getCity(restQry) == null && getCity(restQryNoFilter) == null) {
                queryBean.setAdjustQry(restQry);
            } else {
                queryBean.setQryOnlyCityTags(true);
            }
```



### 重新分词adjustQuery





### 数据召回与初步排序

http://wiki.corp.tujia.com/confluence/pages/viewpage.action?pageId=16584239

```
Queue<SuggestSolrBean> convergedQueue = suggestRecallService.recallAndConverge(context);
```
