
![1594966842702](C:\Users\lelec_1.TUJIA\AppData\Roaming\Typora\typora-user-images\1594966842702.png)

dao服务接口？



"/suggestion/index2"

com.tujia.tns.suggestion.web.controller.SuggestionController.suggestIndex2(@RequestBody BaseRequest<SuggestionParameter> dataRequest)



# suggest业务逻辑

http://wiki.corp.tujia.com/confluence/pages/viewpage.action?pageId=29798834

com.tujia.tns.suggestion.service.impl.SuggestionServiceImpl.suggest(SuggestionParameter parameter)

### 参数验证

![1595209927171](C:\Users\lelec_1.TUJIA\AppData\Roaming\Typora\typora-user-images\1595209927171.png)

[SuggestionNew接口文档](http://wiki.corp.tujia.com/confluence/pages/viewpage.action?pageId=29800441)：

#### suggest()入参SuggestionParameter

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
##包括部分非入参，如下
private String rebuildWrapperId;
@ApiModelProperty(value = "guesslike接口专用字段")
private int typeGroupCount;
@ApiModelProperty(value =a "是否展示加权明细")
private boolean debug;
@ApiModelProperty(value = "前端A/B分桶参数", example = "")
private Map<String, String> abTest;
@ApiModelProperty("判断是否只返回已分销、接入的Ctrip数据，默认为false")
private boolean forCtripHotel = false;
@ApiModelProperty("是否支持crn，若为true，则支持6月版新样式，默认为false")
private boolean forCrn = false;
```

#### suggest()返回参数SuggestionResult

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

- #### SuggestionResults推荐列表

http://wiki.corp.tujia.com/confluence/pages/viewpage.action?pageId=29800441





### 入参信息修改

<img src="C:\Users\lelec_1.TUJIA\AppData\Roaming\Typora\typora-user-images\1595487568570.png" alt="1595487568570" style="zoom: 67%;" />

> ```java
> SourceCodeEnum sourceCodeEnum = SourceCodeEnum.findByCode(parameter.getSourceCode());
> ```
>
> SourceCodeEnum对应不同的渠道![1595487740561](C:\Users\lelec_1.TUJIA\AppData\Roaming\Typora\typora-user-images\1595487740561.png)

- wrapperId处理
- 携程Hotel处理

### 上下文信息保存

将入参信息保存到SuggestionContext对象和QueryBean对象中

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

> 构造方法：![1595489291208](C:\Users\lelec_1.TUJIA\AppData\Roaming\Typora\typora-user-images\1595489291208.png)

```

```



### 是否使用新的展示类型

Apollo中配置：？？

![1595489453320](C:\Users\lelec_1.TUJIA\AppData\Roaming\Typora\typora-user-images\1595489453320.png)



### 噪声过滤与数据预处理

```java
queryAdjustmentService.adjust(context);
context.getQueryBean().setSuggestionContext(context);
```
<img src="C:\Users\lelec_1.TUJIA\AppData\Roaming\Typora\typora-user-images\1595489665835.png" alt="1595489665835" style="zoom:67%;" />

```java
public void adjust(SuggestionContext context) {    
    if (context == null) {        return;    }    
    for (AbstractQueryAdjustFilter filter : filters) {        			    		filter.adjust(context);    
}}
```

| QueryTrimFilter       | 去除所有的空白格                                             |
| --------------------- | ------------------------------------------------------------ |
| PuncReplaceFilter     | 清除特殊符号，包括标点字符、符号（比如数学符号、货币符号等）、分隔符（比如空格、换行等） |
| TraditionalFilter     | 繁体字转换                                                   |
| UselessSuffixFilter   | 无意义后缀，包括附近、周边、周围、租房                       |
| DestinationTrimFilter | 获取用户输入的城市或省份                                     |
| QueryLengthFilter     | 用户输入长度控制，中文及混拼最长50，全为拼音最长150，超过则截取前面内容 |

### 分词

<img src="C:\Users\lelec_1.TUJIA\AppData\Roaming\Typora\typora-user-images\1595490542052.png" alt="1595490542052" style="zoom:80%;" />

<img src="C:\Users\lelec_1.TUJIA\AppData\Roaming\Typora\typora-user-images\1595212253255.png" alt="1595212253255" style="zoom:50%;" />

### 意图识别

> **设置queryBean的各种属性**

```
intentExtractService.extract(context);
```

```
DefaultExtractStrategy.extract(SuggestionContext context);
```

```java
public void extract(SuggestionContext context) {
        QueryBean queryBean = context.getQueryBean();
        // 从原始SuggestionParameter中获取城市id和name
        Integer cityId = context.getOriginPara().getCityId();
        String cityName = context.getOriginPara().getCityName();
        //噪声处理后的query内容
        String adjustQuery = queryBean.getAdjustQry();//王府井做饭沐浴上海
        DestinationInfo destinationInfo = null;
 
```

#### 二框idname方式获取城市信息

```java
//属于二框&&cityId不为空-->通过cityId查询
        if (cityId != null && cityId > 0 && PositionTypeEnum.TWO == context.getQueryBean().getPositionType()) {
            destinationInfo = solrDestinationService.findByDestId(cityId);
            if (destinationInfo == null) {
                destinationInfo = solrDestinationService.loadOneDestFromSolr(cityId, context);
            }
            if (destinationInfo == null) {
                throw new InvalidParameterException("不能被识别的参数cityId:" + cityId);
            }
        }
        //属于二框&&cityName不为空-->使用cityName查询
        if (destinationInfo == null && StringUtils.isNotBlank(cityName)
                && PositionTypeEnum.TWO == context.getQueryBean().getPositionType()) {
            destinationInfo = solrDestinationService.findByDestName(cityName);
            if (destinationInfo == null) {
                CityLocationInfo cityLocationInfo = LoadCityProvinceData.getCity(cityName);
                if (cityLocationInfo != null && StringUtils.equalsIgnoreCase(cityLocationInfo.getCountry(), "中国")) {
                    destinationInfo = solrDestinationService.findByDestName(cityLocationInfo.getName());
                }
            }
            if (destinationInfo == null) {
                throw new InvalidParameterException("不能被识别的参数cityName:" + cityName);
            }
        }

        // 查询到目的地信息则保存至queryBean中
        if (destinationInfo != null) {
            queryBean.setQCity(destinationInfo.getCityName());
            queryBean.setQCityId(destinationInfo.getCityId());
        }
//一二框共用获取城市信息
```



#### 一二框分词方式获取城市信息

```java

        List<String> queryWords = queryBean.getQueryWords();
        {
            // 从分词结果中找城市名
            Set<String> cityNameSet = Sets.newHashSet();//保存筛选过的城市
            queryWords.forEach(word -> {
                //这里可以筛选出城市名，过滤掉无意义的分词
                CityLocationInfo cityLocationInfo = LoadCityProvinceData.getCity(word);
                if (cityLocationInfo != null) {
                    DestinationInfo qDestinationInfo = solrDestinationService
                        .findByDestName(cityLocationInfo.getName());
                    if (qDestinationInfo == null) {//DestinationInfo:{cityId=23,cityName=上海,latitude=31.236447,longitude=121.4802}
                        return;
                    }
                    // queryBean.setQCity(cityLocationInfo.getName());
                    //设置城市名字/id/shortCity？？-->QueryDestination(province=null, shortProvince=null, city=上海, cityId=23, shortCity=上海, subCity=null, shortSubCity=null)
                    queryBean.createQDestinationIfAbsent().setCity(word);//上海
                    queryBean.createQDestinationIfAbsent().setCityId(qDestinationInfo.getCityId());//23
                    queryBean.createQDestinationIfAbsent().setShortCity(cityLocationInfo.getName());//上海
                    cityNameSet.add(word);
                } else if (!Strings.isChineseSearch(word)) {
                    //处理solr中有的城市，在file中没有并解决搜索拼音进入不了城市场景
                    DestinationInfo qDestinationInfo = solrDestinationService.findByDestName(word);
                    if (qDestinationInfo == null) {
                        return;
                    }
                    // queryBean.setQCity(cityLocationInfo.getName());
                    String tempCityName = qDestinationInfo.getCityName();
                    queryBean.createQDestinationIfAbsent().setCity(tempCityName);
                    queryBean.createQDestinationIfAbsent().setCityId(qDestinationInfo.getCityId());
                    queryBean.createQDestinationIfAbsent().setShortCity(tempCityName);
                    cityNameSet.add(word);
                }
            });

            // 用户输入为城市时且为一框逻辑时，重写query-->(王府井,上海,做饭,沐浴)
            Set<String> queryWordSet = new HashSet<>(queryWords);
            //移除已经筛选过的城市名(这个城市名保存在QueryDestination了)
            queryWordSet.removeAll(cityNameSet);//-->(王府井,做饭,沐浴)
            //如果是一框&&都已经筛选过，设置查询词adjustqry
            if (CollectionUtils.isEmpty(queryWordSet) && queryBean.getPositionType() == PositionTypeEnum.ONE) {
                queryBean.setAdjustQry(queryBean.createQDestinationIfAbsent().getShortCity());
            }

            // 如果没有获取到城市信息，则从分词结果中找乡镇名对应城市
            if (StringUtils.isBlank(queryBean.getQCity())) {
                queryWords.forEach(word -> {//   queryWords.size=4
                    if (chinaSubCity.get(word) != null) {//子城市
                        // queryBean.setQCity(chinaSubCity.get(word));
                        queryBean.createQDestinationIfAbsent().setSubCity(word);
                        queryBean.createQDestinationIfAbsent().setShortSubCity(word);
                    } else if (chinaSubCityFull.get(word) != null) {
                        // queryBean.setQCity(chinaSubCity.get(chinaSubCityFull.get(word)));
                        queryBean.createQDestinationIfAbsent().setSubCity(word);
                        queryBean.createQDestinationIfAbsent().setShortSubCity(chinaSubCityFull.get(word));
                    }
                });
            }
        }
        //至此可以获取到城市信息
        String cityInQuery = queryBean.createQDestinationIfAbsent().getShortCity();

        // 从坐标查位置城市
        DestinationInfo bCityDest = coordCityService.getNearestCity(context.getOriginPara().getCoord());
        if (bCityDest != null) {
            queryBean.setBCity(bCityDest.getCityName());
        }
//一二框获取省份信息
       
```

#### 一二框获取省份信息

```java
 // 获取省份信息
        queryWords.forEach(word -> {
            ProvinceLocationInfo provinceLocationInfo = LoadCityProvinceData.getProvince(word);
            // 从分词结果找省份
            if (provinceLocationInfo != null && !queryBean.hasQDestinationProvince()) {
                queryBean.setProvince(provinceLocationInfo.getName());
                queryBean.createQDestinationIfAbsent().setProvince(word);
                queryBean.createQDestinationIfAbsent().setShortProvince(provinceLocationInfo.getName());

                // Qcity对应省份
            } else if (StringUtils.isNotBlank(queryBean.getQCity())) {
                String qCity = queryBean.getQCity();
                if (chinaCity.get(qCity) != null && StringUtils.isNotBlank(chinaCity.get(qCity).getProvince())) {
                    queryBean.setProvince(chinaCity.get(qCity).getProvince());
                } else if (pyChinaCity.get(qCity) != null
                        && StringUtils.isNotBlank(pyChinaCity.get(qCity).getProvince())) {
                    queryBean.setProvince(pyChinaCity.get(qCity).getProvince());
                }
            }
        });
   
```



#### 一框二框提取特色场景

```java
 // tag筛选项
        Map<String, String> feaTagRestQMap = getFeatureTagsRestQry(context);
// 提取结果保存至queryBean中
        if (feaTagRestQMap != null) {
            String jsonStr = feaTagRestQMap.get("tagList");//[{"featureTagName":"能做饭的{city}民宿","featureTagValue":301,"featureTagOldValue":null,"featureTagOldConditionType":null,"featureTagOldConditionTypeName":null,"valid":true,"alias":["可做饭","做饭","厨房","能做饭的{city}民宿"],"type":3,"typeName":null,"code":1,"webapiValid":true}]
            List<FeatureTag> featureTagList = JsonUtils.readValue(jsonStr, new TypeReference<List<FeatureTag>>() {
            });
            queryBean.setFeatureTagList(featureTagList);

            String jsonNameStr = feaTagRestQMap.get("tagNameList");//["做饭方便"]
            List<String> tags = JsonUtils.readValue(jsonNameStr, new TypeReference<List<String>>() {
            });
            queryBean.setTags(tags);

            // 用户输入除去特色后的内容
            String restQry = feaTagRestQMap.get("restQry");
            String restQryNoFilter = context.getQueryBean().getAdjustQryNoFilter();//王府井沐浴上海
            // 剩余内容不能为目的地
            if (StringUtils.isNotBlank(restQry) && getCity(restQry) == null && getCity(restQryNoFilter) == null) {
                queryBean.setAdjustQry(restQry);
            } else {
                queryBean.setQryOnlyCityTags(true);
            }
        }
```

![1595496163667](C:\Users\lelec_1.TUJIA\AppData\Roaming\Typora\typora-user-images\1595496163667.png)


```java
private Map<String, String> getFeatureTagsRestQry(SuggestionContext context) {
        String initQry = context.getQueryBean().getAdjustQry();
        String wrapperId = context.getOriginPara().getRebuildWrapperId();
        context.getQueryBean().setAdjustQryUnext(initQry);
        // 初始化
        context.getQueryBean().setAdjustQryNoFilter(initQry);
        if (!Strings.isChineseSearch(initQry)) {
            List<String> pinYinList = PinYinHandler.getPinyinList(initQry);
            if (CollectionUtils.isNotEmpty(pinYinList)) {
                context.getQueryBean().setAdjustQryUnextPy(pinYinList.get(0));
                context.getQueryBean().setAdjustQryNoFilterPy(pinYinList.get(0));
            }
        }

        // 留作后续判断原始用户输入是否变化
        String initQryBak = initQry;
        String restQry = initQry;
        List<String> queryWords = context.getQueryBean().getQueryWords();
        // Apollo中的featureTag列表
        Map<String, FeatureTag> featureTagMap;
        Map<String, FeatureTag> pyFeatureTagMap;
        SuggestionConfig suggestionConfig = suggestionConfigCache.getSearchControlConfig();
        // 筛选项标准名映射
        Map<Integer, String> featureTagNameMap = suggestionConfig.getFeatureTagNameMap();
        //这里通过wrapperId获取不同渠道的的特色列表
        if (StringUtils.equalsIgnoreCase(wrapperId, EnumWrapperId.WAPTUJIA016.getDesc())) {
            featureTagMap = suggestionConfig.getQunarFeatureTagMap();
            pyFeatureTagMap = suggestionConfig.getPyQunarFeatureTagMap();
        } else if (StringUtils.equalsIgnoreCase(wrapperId, EnumWrapperId.WAP_MAYIBNB.getDesc())) {
            featureTagMap = suggestionConfig.getMayiFeatureTagMap();
            pyFeatureTagMap = suggestionConfig.getPyMayiFeatureTagMap();
        } else if (StringUtils.equalsIgnoreCase(wrapperId, EnumWrapperId.WAP_CTRIPBNB.getDesc())) {
            featureTagMap = suggestionConfig.getCtripFeatureTagMap();
            pyFeatureTagMap = suggestionConfig.getPyCtripFeatureTagMap();
        } else {
            featureTagMap = suggestionConfig.getTujiaFeatureTagMap();
            pyFeatureTagMap = suggestionConfig.getPyTujiaFeatureTagMap();
        }
        // 不关心queryWordsPy是否为空；没有特色
        if (CollectionUtils.isEmpty(queryWords) || featureTagMap.isEmpty() || pyFeatureTagMap.isEmpty()) {
            return null;
        }
        // 用户输入内容中的特色集合
        List<FeatureTag> tagList = Lists.newArrayList();
        List<String> tagNameList = Lists.newArrayList();
        // 特色去重
        Set<String> tagSet = Sets.newHashSet();
        FeatureTag tag;
        String stdTagName;
        // 1.特色提取-从分词结果中提取
        restQry = restQry.toLowerCase();
        for (String word : queryWords) {
            word = word.toLowerCase();
            if (featureTagMap.containsKey(word)) {
                tag = featureTagMap.get(word);
                stdTagName = featureTagNameMap.get(tag.getFeatureTagValue());

                // 非Ctrip（携程）四居筛选项特殊识别处理
                if (!EnumWrapperId.WAP_CTRIPBNB.getDesc().equalsIgnoreCase(wrapperId)
                        && tag.getFeatureTagValue() == EnumSearchHouseModel.Four.getLabelKey()) {
                    // 若用户输入为五-八居，则改为四居
                    // NOTE 需要修改用户输入，在下面还有回写的操作
                    restQry = restQry.replaceAll(word, StaticConstants.FOUR_RESIDENCE);
                    initQry = initQry.replaceAll(word, StaticConstants.FOUR_RESIDENCE);
                }

                if (!tagSet.contains(stdTagName)) {
                    tagList.add(tag);
                    tagNameList.add(stdTagName);
                }
                tagSet.add(stdTagName);
                restQry = restQry.replaceFirst(word, "");
            }
        }

        // 2.特色提取-拼音处理
        if (!Strings.isChineseSearch(restQry)) {
            String tempQry = "";
            // 注意queryBean.getQueryWordsPy()字段中的内容为单字拼音
            List<String> pinYinList = PinYinHandler.getPinyinList(restQry);
            if (CollectionUtils.isNotEmpty(pinYinList)) {
                tempQry = pinYinList.get(0).toLowerCase();
            }
            String tagName;
            for (Map.Entry<String, FeatureTag> entry : pyFeatureTagMap.entrySet()) {
                if (tempQry.contains(tagName = entry.getKey())) {
                    tag = pyFeatureTagMap.get(tagName);
                    stdTagName = featureTagNameMap.get(tag.getFeatureTagValue());
                    if (!tagSet.contains(stdTagName)) {
                        tagList.add(tag);
                        tagNameList.add(stdTagName);
                    }
                    tagSet.add(stdTagName);
                    tempQry = tempQry.replace(tagName, "");
                    restQry = restQry.replace(tagName, "");
                }
            }

            // 判断通过拼音去除特色后的内容，与原剩余内容去除特色后的拼音内容，二者是否一致，若一致则说明去除所有特色的剩余内容可用中文表示
            pinYinList = PinYinHandler.getPinyinList(restQry);
            if (CollectionUtils.isNotEmpty(pinYinList)) {
                String restQryPy = pinYinList.get(0);
                if (!StringUtils.equalsIgnoreCase(restQryPy, tempQry)) {
                    restQry = tempQry;
                }
            }
        }
        // 保存除去筛选项后的剩余内容
        context.getQueryBean().setAdjustQryNoFilter(restQry);
        if (StringUtils.isNotBlank(context.getQueryBean().getAdjustQryNoFilterPy())) {
            List<String> pinYinList = PinYinHandler.getPinyinList(restQry);
            if (CollectionUtils.isNotEmpty(pinYinList)) {
                context.getQueryBean().setAdjustQryNoFilterPy(pinYinList.get(0));
            }
        }

        if (CollectionUtils.isEmpty(tagList)) {
            return null;
        }

        // 未提取时用户输入过长（中文及混拼超过10，全为拼音超过20），不删除提取出的筛选项内容
        int qryLen = StringUtils.length(initQry);
        boolean isQueryTooLong = false;
        if ((Strings.isPinyinSearch(initQry) && qryLen >= StaticConstants.QUERY_PINYIN_LEN_MAX)
                || qryLen >= StaticConstants.QUERY_MIX_LEN_MAX) {
            isQueryTooLong = true;
        }
        // 剩余内容过短，对召回及排序影响比较大，易丢失关键信息，不删除提取出的筛选项内容
        if (StringUtils.length(restQry) <= 2 || isQueryTooLong) {
            restQry = initQry;
        }

        // NOTE 用户输入在提取特色前后对比，主要是非Ctrip渠道关于户型的处理，需要覆盖原用户输入
        if (!StringUtils.equalsIgnoreCase(initQry, initQryBak)) {
            log.info("origin user input is {}, but {} instead now", initQryBak, initQry);
            context.getQueryBean().setAdjustQry(initQry);
            context.getQueryBean().setAdjustQryUnext(initQry);
        }


        Map<String, String> feaTagRestQMap = Maps.newHashMap();
        feaTagRestQMap.put("tagList", JsonUtils.writeValueAsString(tagList));
        feaTagRestQMap.put("tagNameList", JsonUtils.writeValueAsString(tagNameList));
        feaTagRestQMap.put("restQry", restQry);
        return feaTagRestQMap;
    }
```

#### 重写query

```java
 // 根据分词结果判断
        // 用户输入命中省级单位且结尾为省或区，且输入到市级，如“湖北省武汉”
        if (StringUtils.isNotBlank(queryBean.createQDestinationIfAbsent().getProvince())
                && (queryBean.createQDestinationIfAbsent().getProvince().endsWith("省")
                || queryBean.createQDestinationIfAbsent().getProvince().endsWith("区"))
                && StringUtils.isNotBlank(queryBean.createQDestinationIfAbsent().getCity())) {
            queryBean.setIntentCity(cityInQuery);
            // 如果与QCity一致，则对query重写
            String originAdjustQry = queryBean.getAdjustQry();
            String remProvince = originAdjustQry.replaceAll(queryBean.createQDestinationIfAbsent().getProvince(), "");
            String remProvinceAndCity = remProvince.replaceAll(queryBean.createQDestinationIfAbsent().getCity(), "");
            if (positionType == PositionTypeEnum.TWO) {
                if (StringUtils.isBlank(remProvinceAndCity)) {
                    queryBean.setAdjustQry(queryBean.getQDestination().getShortCity());
                } else if (StringUtils.equalsIgnoreCase(cityInQuery,
                        queryBean.getQCity())) {
                    queryBean.setAdjustQry(remProvinceAndCity);
                }
            } else if (positionType == PositionTypeEnum.ONE) {
                if (StringUtils.isBlank(remProvinceAndCity)) {
                    queryBean.setAdjustQry(queryBean.getQDestination().getShortCity());
                }
            }
        }
        // 用户输入了某区县级城市(包括直辖市)且结尾为市/区/州/县等且市级不为空，如“武汉（市）黄陂区”
        else if (StringUtils.isNotBlank(queryBean.createQDestinationIfAbsent().getCity())
                && StringUtils.isNotBlank(queryBean.createQDestinationIfAbsent().getSubCity())
                && (queryBean.createQDestinationIfAbsent().getSubCity().endsWith("市")
                || queryBean.createQDestinationIfAbsent().getSubCity().endsWith("州")
                || queryBean.createQDestinationIfAbsent().getSubCity().endsWith("区")
                || queryBean.createQDestinationIfAbsent().getSubCity().endsWith("县"))) {
            queryBean.setIntentCity(cityInQuery);
            // 如果与QCity一致，则对query重写
            String originAdjustQry = queryBean.getAdjustQry();
            String remCity = originAdjustQry.replaceAll(queryBean.createQDestinationIfAbsent().getCity(), "");
            if (positionType == PositionTypeEnum.TWO) {
                if (StringUtils.equalsIgnoreCase(cityInQuery,
                        queryBean.getQCity())) {
                    queryBean.setAdjustQry(remCity);
                }
            }
        }
        // 用户输入了某市级城市(包括直辖市)且结尾为市/区/州/县等，如“武汉市”
        else if (StringUtils.length(queryBean.createQDestinationIfAbsent().getCity()) >= 3
                && (queryBean.createQDestinationIfAbsent().getCity().endsWith("市")
                || queryBean.createQDestinationIfAbsent().getCity().endsWith("州")
                || queryBean.createQDestinationIfAbsent().getCity().endsWith("区")
                || queryBean.createQDestinationIfAbsent().getCity().endsWith("县"))) {
            queryBean.setIntentCity(cityInQuery);
            // 如果与QCity一致，则对query重写
            String originAdjustQry = queryBean.getAdjustQry();
            String remCity = originAdjustQry.replaceAll(queryBean.createQDestinationIfAbsent().getCity(), "");
            if (positionType == PositionTypeEnum.TWO && StringUtils.isNotBlank(remCity)) {
                if (StringUtils.equalsIgnoreCase(cityInQuery,
                        queryBean.getQCity())) {
                    queryBean.setAdjustQry(remCity);
                }
            }
        }
```

#### 一框提取城市场景

```java
// 提取场景
        if (positionType == PositionTypeEnum.ONE) {
            QrySceneEnum qrySceneEnum = null;
            // province场景
            String province = null;
            boolean provinceScene = true;
            // 判断是否为海外province，若是则不处理【省】，如巴厘省
            for (String word : queryWords) {
                boolean isHaiwaiProvince = false;
                // 从分词结果找省份
                if (chinaProvince.containsKey(word) || pyChinaProvince.containsKey(word)
                        || (isHaiwaiProvince = (foreignProvince.containsKey(word) || pyForeignProvince.containsKey(word)))) {
                    province = word;
                    if (province.endsWith("省") && !isHaiwaiProvince) {
                        province = province.substring(0, province.length() - 1);
                    }
                } else {
                    provinceScene = false;
                }
            }

            if (provinceScene && province != null) {
                qrySceneEnum = QrySceneEnum.Province;
                queryBean.setAdjustQry(province);
            } else {
                String adjustQry = queryBean.getAdjustQry();
                if (countrySet.contains(adjustQry)) {// 国家场景
                    qrySceneEnum = QrySceneEnum.County;
                } else {// 城市场景
                    String cityContent = "";
                    boolean isHaiwaiCity = false;
                    SceneResult chinaCitySceneResult = null;
                    SceneResult foreignCitySceneResult = null;
                    // 城市场景命中次数，若用户输入为相同名称的国内和海外城市（如泰安），则不进入城市场景
                    int pointCitySenceCount = 0;

                    if (StringUtils.isNotBlank(cityInQuery)) {
                        chinaCitySceneResult = solrDestinationService.isChinaCity(cityInQuery);
                        if (chinaCitySceneResult.isScene()) {
                            cityContent = cityInQuery;
                            pointCitySenceCount++;
                        }
                        foreignCitySceneResult = solrDestinationService.isForeignCity(cityInQuery);
                        if (foreignCitySceneResult.isScene()) {
                            cityContent = cityInQuery;
                            isHaiwaiCity = true;
                            pointCitySenceCount++;
                        }
                    }

                    // 只处理用户输入与目的地名称完全一致的情况
                    if (pointCitySenceCount != 2 && cityContent.length() > 0 && (StringUtils.equalsIgnoreCase(adjustQry, cityContent)
                            || StringUtils.equalsIgnoreCase(adjustQry, cityContent + "市"))) {
                        if (isHaiwaiCity) {
                            qrySceneEnum = QrySceneEnum.ForeignCity;
                            String foreignCityName = foreignCitySceneResult.getCityName();
                            if (foreignCityName != null) {
                                adjustQry = foreignCityName;
                            }
                        } else {
                            qrySceneEnum = QrySceneEnum.ChinaCity;
                            if (chinaCitySceneResult != null && StringUtils.isNotBlank(chinaCitySceneResult.getCityName())) {
                                adjustQry = chinaCitySceneResult.getCityName();
                            }
                        }
                    }

                    queryBean.setAdjustQry(adjustQry);
                }
            }
            queryBean.setScene(qrySceneEnum);
        }
    }
```

##### 意图识别后的querybean属性

````java
public class QueryBean {
    private SuggestionParameter originPara;
    /** query经过trim/噪声等处理，以及经意图识别后的内容 */
    private String adjustQry;//昌平做饭浴缸三居北京
    /** query经过trim/噪声等处理，以及经意图识别后除去筛选项后的剩余内容 */
    private String adjustQryNoFilter;//昌平北京
    /** query经过trim/噪声等处理，未经意图识别后的内容 */
    private String adjustQryUnext;//昌平做饭浴缸三居北京
    private String adjustQryPy;
    private String adjustQryUnextPy;
    private String adjustQryNoFilterPy;

    /** adjustQry分词结果 */
    private List<String> queryWords;// size = 5
    private Map<String, Double> queryTermIDF;
    private Map<String, Double> queryTermIDFWeight;

    private List<String> queryWordsPy;

    /** 意图识别信息 */
    private String bCity;
    private String qCity;
    private Integer qCityId;
    private String province;
    /** 二框中用户可能想切换的城市 */
    private String intentCity;
    /** query中包含的各级目的地 */
    private QueryDestination qDestination;//QueryDestination(province=null, shortProvince=null, city=北京, cityId=48, shortCity=北京, subCity=昌平, shortSubCity=昌平)
    /** 去除了用户输入中的省份城市信息 */
    private QueryAnalysisInfo rmIntentDestQryInfo;
    private QrySceneEnum scene;
    /** 仅记录根据用户输入匹配到featureTag的标准名 */
    private List<String> tags;
    /** 记录featureTag信息 */
    private List<FeatureTag> featureTagList;
    /** 用户输入只包含城市或特色 */
    private boolean qryOnlyCityTags;

    private SuggestionContext suggestionContext;
    Map<String, Double> queryStringWeight;

    /** 0-不分框 1-分框:1框 2-分框：2框 */
    private PositionTypeEnum positionType = PositionTypeEnum.NONE;

    private InternalOverseasEnum internalOverseas;

    /**
     * 若为本站渠道，判断版本号是否>=206，用于后续对本站高版本的功能进行控制
     */
    private boolean tujiaVersionOver206 = false;
````





### 重新分词adjustQuery

```java
 //重新分词，adjustQuery可能发生了变化
        adjustQry = context.getQueryBean().getAdjustQry();
        adjustQry = StringUtils.lowerCase(adjustQry);
        if (StringUtils.isNotBlank(adjustQry)) {
            //提取特色，场景后剩余信息
            List<String> words = tokenizerService.tokenize(adjustQry);
            context.getQueryBean().setQueryWords(words);

            String adjustQryPy = "";
            List<String> pinYinList = PinYinHandler.getPinyinList(adjustQry);
            if (CollectionUtils.isNotEmpty(pinYinList)) {
                adjustQryPy = pinYinList.get(0);
            }
            context.getQueryBean().setAdjustQryPy(adjustQryPy);

            List<String> wordsPy = PinyinSegment.getPinyinSegmentNoWhiteSpaces(adjustQryPy);// {wang,fu,jing,zuo,fan,....}
            context.getQueryBean().setQueryWordsPy(wordsPy);

            // idf处理
            queryWordsIDFHandle(context, words);
        }
```



```java
//如果用户输入中包含了省份、城市信息;剩余信息如果有，要加入分词集中进行solr查询
        if (context.getQueryBean().hasQDestination()) {
            String rmIntentDestQry = context.getQueryBean().rmIntentCityProvinceQry();//昌平做饭浴缸三居(移除了北京)
            if (StringUtils.isNotBlank(rmIntentDestQry) && !StringUtils.equalsIgnoreCase(rmIntentDestQry, adjustQry)) {
                QueryAnalysisInfo rmIntentAnalysisInfo = new QueryAnalysisInfo();
                rmIntentAnalysisInfo.setAdjustQry(rmIntentDestQry);
                List<String> words = tokenizerService.tokenize(rmIntentDestQry);
                rmIntentAnalysisInfo.setQueryWords(words);

                String adjustQryPy = "";
                //汉字转拼音集合
                List<String> pinYinList = PinYinHandler.getPinyinList(rmIntentDestQry);
                if (CollectionUtils.isNotEmpty(pinYinList)) {
                    adjustQryPy = pinYinList.get(0);
                }
                rmIntentAnalysisInfo.setAdjustQryPy(adjustQryPy);

                List<String> wordsPy = PinyinSegment.getPinyinSegmentNoWhiteSpaces(adjustQryPy);
                rmIntentAnalysisInfo.setQueryWordsPy(wordsPy);//QueryAnalysisInfo(adjustQry=昌平做饭浴缸三居, adjustQryPy=changpingzuofanyugangsanju, queryWords=[昌平, 做饭, 浴缸, 三居], queryWordsPy=[chang, ping, zuo, fan, yu, gang, san, ju])
                context.getQueryBean().setRmIntentDestQryInfo(rmIntentAnalysisInfo);
            }
        }
```



### 数据召回与初步排序

http://wiki.corp.tujia.com/confluence/pages/viewpage.action?pageId=16584239

```
Queue<SuggestSolrBean> convergedQueue = suggestRecallService.recallAndConverge(context);
```

![1595498527526](C:\Users\lelec_1.TUJIA\AppData\Roaming\Typora\typora-user-images\1595498527526.png)

根据不同的渠道有不同的召回策略

![1595498592451](C:\Users\lelec_1.TUJIA\AppData\Roaming\Typora\typora-user-images\1595498592451.png)

![1595499002040](C:\Users\lelec_1.TUJIA\AppData\Roaming\Typora\typora-user-images\1595499002040.png)

```java
public Queue<SuggestSolrBean> recallAndConverge(SuggestionContext context) {
        long startTime = System.currentTimeMillis();
        Queue<SuggestSolrBean> recalledData = recall(context);
        filter(context, recalledData);
        long recallFinishTime = System.currentTimeMillis();
        Queue<SuggestSolrBean> rtnQueue = converge(recalledData, context);
        long convergeFinishTime = System.currentTimeMillis();
        log.info("recall time:{},converge time:{}", recallFinishTime - startTime, convergeFinishTime - recallFinishTime);
        return rtnQueue;
    }
```



#### SceneRecallStrategy.class

![1595501560544](C:\Users\lelec_1.TUJIA\AppData\Roaming\Typora\typora-user-images\1595501560544.png)

![1595499420697](C:\Users\lelec_1.TUJIA\AppData\Roaming\Typora\typora-user-images\1595499420697.png)

##### 国内国外城市

```java
 private Queue<SuggestSolrBean> citySceneRecall(SuggestionContext context, boolean oversea) {
        Queue<SuggestSolrBean> suggestSolrBeanResult = Lists.newLinkedList();
        //召回相同城市热门景区和地标
        CitySceneResult citySceneResult = guessULike.citySceneHandle(context, oversea);
        List<SuggestSolrBean> suggestSolrBeans = solrRetrieveComponent
                .retrieveDestinationOrLocation(context.getQueryBean().getAdjustQry(), context, oversea);
        //进行排序
        defaultRankComponent.weightPrehandle(suggestSolrBeans, context);
        //对行政区降权
        suggestSolrBeans.stream().forEach(vo -> {
            if (vo.getConditionType() == ConditionTypeEnum.LOCATION
                    && vo.getSecondConditionType() == SecondConditionTypeEnum.DISTRICT) {
                vo.setWeight(vo.getWeight() * 0.7);
            }
        });
 //排序后选择和目的地名字相同的第一个加入推荐列表？？？？？？
        suggestSolrBeans = suggestSolrBeans.stream().sorted().collect(Collectors.toList());
        String destinationName = context.getQueryBean().getAdjustQry();
        Optional<SuggestSolrBean> first = suggestSolrBeans.stream().filter(bean -> bean.isNameEquals(destinationName))
                .findFirst();
        SuggestSolrBean destinationSolrBean = null;
        int childGroupSize = 0;
        Set<String> itemInfoSet = Sets.newHashSet();
        if (first.isPresent()) {
            destinationSolrBean = first.get();
            //获取guessULike热门信息分组列表，以及少于热门推荐等最小数量的列表
            Map<String, Queue<SuggestSolrBean>> retMap = getChildGroupLessQueue(citySceneResult);
            Queue<SuggestSolrBean> childGroups = retMap.get(CHILD_GROUP_QUEUE);
            Queue<SuggestSolrBean> hotLessQueue = retMap.get(HOT_LESS_QUEUE);
            if (childGroups != null && childGroups.size() > 0) {
                childGroupSize = childGroups.size();
                //将热门景点推荐加入到第一个推荐bean中？？？？？？？？？
                destinationSolrBean.setChildGroups(childGroups);
                itemInfoSet = Sets.newHashSet(citySceneResult.getItemInfoSet());
            }
            //添加到推荐列表
            suggestSolrBeanResult.add(destinationSolrBean);
            // 对少于热门推荐等最小数量的queue的处理-->加入到召回结果集
            if (CollectionUtils.isNotEmpty(hotLessQueue)) {
                suggestSolrBeans.addAll(hotLessQueue);
                suggestSolrBeans = suggestSolrBeans.stream().sorted().collect(Collectors.toList());
                if (CollectionUtils.isNotEmpty(itemInfoSet)) {
                    for (SuggestSolrBean vo : hotLessQueue) {
                        itemInfoSet.remove(SuggestItemUtil.createItemInfo(vo));
                    }
                }
            }
        }
        //遍历其余的召回结果，一共15条数据返回
        for (SuggestSolrBean suggestSolrBean : suggestSolrBeans) {
            String itemInfo = SuggestItemUtil.createItemInfo(suggestSolrBean);
            if (itemInfoSet.contains(itemInfo) || suggestSolrBean.equals(destinationSolrBean)) {
                continue;
            }

            suggestSolrBeanResult.add(suggestSolrBean);
            itemInfoSet.add(itemInfo);
            if (suggestSolrBeanResult.size() + childGroupSize >= StaticConstants.RETRIEVE_MAX_NUM) {
                break;
            }
        }

        return suggestSolrBeanResult;
    }
```



```java
//获取热门信息分组列表，以及少于热门推荐等最小数量的列表
private Map<String, Queue<SuggestSolrBean>> getChildGroupLessQueue(CitySceneResult citySceneResult) {
        Map<String, Queue<SuggestSolrBean>> retMap = Maps.newHashMap();
        Queue<SuggestSolrBean> hotLessQueue = Lists.newLinkedList();
        Queue<SuggestSolrBean> childGroups = Lists.newLinkedList();
        SuggestSolrBean viewSpotSuggestSolrBean = citySceneResult.getViewSpotSuggestSolrBean();
        SuggestSolrBean landmarkSuggestSolrBean = citySceneResult.getLandmarkSuggestSolrBean();
        SuggestSolrBean houseTypeSuggestionBean = citySceneResult.getHouseTypeSuggestionBean();
        if (viewSpotSuggestSolrBean != null) {
            if (viewSpotSuggestSolrBean.getChildGroups().size() >= StaticConstants.RETRIEVE_HOT_MIN_NUM) {
                childGroups.add(viewSpotSuggestSolrBean);
            } else {
                hotLessQueue.addAll(viewSpotSuggestSolrBean.getChildGroups());
            }
        }
        if (landmarkSuggestSolrBean != null) {
            if (landmarkSuggestSolrBean.getChildGroups().size() >= StaticConstants.RETRIEVE_HOT_MIN_NUM) {
                childGroups.add(landmarkSuggestSolrBean);
            } else {
                hotLessQueue.addAll(landmarkSuggestSolrBean.getChildGroups());
            }
        }
        if (houseTypeSuggestionBean != null) {
            if (houseTypeSuggestionBean.getChildGroups().size() >= StaticConstants.RETRIEVE_HOT_MIN_NUM) {
                childGroups.add(houseTypeSuggestionBean);
            } else {
                hotLessQueue.addAll(houseTypeSuggestionBean.getChildGroups());
            }
        }
        retMap.put(CHILD_GROUP_QUEUE, childGroups);
        retMap.put(HOT_LESS_QUEUE, hotLessQueue);
        return retMap;
    }
```

##### 省份或国家

```java
    private Queue<SuggestSolrBean> provinceOrCountry(SuggestionContext context) {
        Queue<SuggestSolrBean> resultList = Lists.newLinkedList();
        Set<String> itemInfoSet = Sets.newHashSet();
        QueryBean queryBean = context.getQueryBean();
        String adjustQry = queryBean.getAdjustQry();
        QrySceneEnum scene = queryBean.getScene();
        String q = "exactFields:" + adjustQry;
        String suffix;
        if (scene.equals(QrySceneEnum.Province)) {
            suffix = "_province";
        } else {
            suffix = "_country";
        }
        q = q + suffix;
        List<SuggestSolrBean> prefixResultList = solrRetrieveComponent.retrieveProvinceOrCountry(q, context);
        defaultRankComponent.weightPrehandle(prefixResultList, context);
        prefixResultList = prefixResultList.stream().sorted().collect(Collectors.toList());
        // 目的地下热门地标
        SuggestSolrBean firstBean = null;
        int childGroupSize = 0;
        if (CollectionUtils.isNotEmpty(prefixResultList)
                && (firstBean = prefixResultList.get(0)).getConditionType() == ConditionTypeEnum.DESTINATION) {
            //获取第一个为目的地的城市id
            int cityId = firstBean.getCityId();
            //提取出城市id相同的所有的 结果集
            List<SuggestSolrBean> firstCityBeanList = prefixResultList.stream().filter(vo -> vo.getCityId() == cityId)
                    .collect(Collectors.toList());
            if (CollectionUtils.isNotEmpty(firstCityBeanList)) {
                boolean oversea = false;
                if (firstBean.getInternalOverseas() == InternalOverseasEnum.Overseas.getCode()) {
                    oversea = true;
                }
                //
                CitySceneResult citySceneResult = guessULike.citySceneHandle(context, oversea, firstCityBeanList, firstBean);
                Map<String, Queue<SuggestSolrBean>> retMap = getChildGroupLessQueue(citySceneResult);
                Queue<SuggestSolrBean> childGroups = retMap.get(CHILD_GROUP_QUEUE);
                Queue<SuggestSolrBean> hotLessQueue = retMap.get(HOT_LESS_QUEUE);
                if (childGroups != null && childGroups.size() > 0) {
                    childGroupSize = childGroups.size();
                    //目的地下所有的推荐结果集
                    firstBean.setChildGroups(childGroups);
                    itemInfoSet = Sets.newHashSet(citySceneResult.getItemInfoSet());
                }
                String itemInfo = SuggestItemUtil.createItemInfo(firstBean);
                itemInfoSet.add(itemInfo);
                //返回结果中添加  目的地（包括子景点）
                resultList.add(firstBean);

                // 对少于热门推荐等最小数量的queue的处理
                if (CollectionUtils.isNotEmpty(hotLessQueue)) {
                    prefixResultList.addAll(hotLessQueue);
                    prefixResultList = prefixResultList.stream().sorted().collect(Collectors.toList());
                    if (CollectionUtils.isNotEmpty(itemInfoSet)) {
                        for (SuggestSolrBean vo : hotLessQueue) {
                            itemInfoSet.remove(SuggestItemUtil.createItemInfo(vo));
                        }
                    }
                }
            }
        }
        // 其余目的地、地标类数据处理
        for (SuggestSolrBean suggestSolrBean : prefixResultList) {
            if (resultList.size() >= StaticConstants.RETRIEVE_MAX_NUM) {
                break;
            }
            String itemInfo = SuggestItemUtil.createItemInfo(suggestSolrBean);
            if (itemInfoSet.contains(itemInfo)) {
                continue;
            }
            resultList.add(suggestSolrBean);
            itemInfoSet.add(itemInfo);
        }

        if (resultList.size() + childGroupSize >= StaticConstants.RETRIEVE_MAX_NUM) {
            return resultList;
        }

        List<SuggestSolrBean> suggestSolrBeanList = solrRetrieveComponent.retrievePrefixResult(queryBean, context);
        defaultRankComponent.weightPrehandle(suggestSolrBeanList, context);
        suggestSolrBeanList = suggestSolrBeanList.stream().sorted().collect(Collectors.toList());
        for (SuggestSolrBean suggestSolrBean : suggestSolrBeanList) {
            if (resultList.size() + childGroupSize >= StaticConstants.RETRIEVE_MAX_NUM) {
                break;
            }
            String itemInfo = SuggestItemUtil.createItemInfo(suggestSolrBean);
            if (itemInfoSet.contains(itemInfo)) {
                continue;
            }
            resultList.add(suggestSolrBean);
            itemInfoSet.add(itemInfo);
        }

        return resultList;
    }
```

#### PositionOneRecallStrategy

> 一框召回策略

##### 中文前缀+中文全文

```java
 //中文前缀+中文全文
        if (Strings.isChineseSearch(trimAdjustQry) || Strings.isChineseSearch(trimAdjustQry.substring(trimAdjustQry.length()-1))) {
            List<Future<List<SuggestSolrBean>>> rmIntentDestFutureList = null;
            //1.如果有目的地信息
            if (hasQDestination && rmIntentDestQryInfo != null) {
                //异步执行前缀召回和全文召回
                rmIntentDestFutureList = removeIntentDestRetrieve(context, rmIntentDestQryInfo, false);
            }
            //执行中文前缀召回
            RetrieveResultSummary chPreRetrieveSummary = chineseRetrieve(SearchTypeEnum.PRE_MATCH, context,
                    context.getQueryBean().getQCityId(), false, 50,
                    true,
                    false);
            Map<String, SuggestSolrBean> chPreRetrieveMap = chPreRetrieveSummary.getAllResults();
            //合并两次召回结果，添加到了chPreRetrieveMap中
            getFutureResult(rmIntentDestFutureList, chPreRetrieveMap);
            // 若到前缀召回处理后的时间过长，或者结果集数量多，则直接将结果返回，避免后续加入全文召回导致前端、中间层等等待超时
            boolean isPreMatchCostMuch = context.isRetrieveCostMuch();
            if (chPreRetrieveSummary.requireCountFullfilled() || isPreMatchCostMuch) {
                if (isPreMatchCostMuch) {
                    long timecost = System.currentTimeMillis() - startTime;
                    TMonitor.recordOne(TMonitorConstants.successOf(TMonitorConstants.PRE_MATCH_COST_MUCH), timecost);
                    log.info("一框中文前缀匹配 {} with timecost:{}ms", TMonitorConstants.PRE_MATCH_COST_MUCH, timecost);
                }
                //结果排序
                return getRankComponent().chnFullText(new ArrayList<>(chPreRetrieveMap.values()), context);
            }
            //不超时，进行全文召回
            RetrieveResultSummary chFullRetrieveSummary = chineseRetrieve(SearchTypeEnum.ALL_MATCH, context,
                    context.getQueryBean().getQCityId(), false, 50,
                    true,
                    false);
            chPreRetrieveMap.putAll(chFullRetrieveSummary.getAllResults());
            List<SuggestSolrBean> unionCollection = new ArrayList<>(chPreRetrieveMap.values());
            return getRankComponent().chnFullText(unionCollection, context);
        }
```

##### 拼音前缀+拼音全文

```java

```

##### 中文、拼音混合，采取中文全文+拼音全文

```java

```

####  CustomizedSceneRecallStrategy 

> 二框搜索"民宿",定制化召回策略

##### 根据单一筛选项召回城市





##### 修改权重

```java
Map<Integer, FeatureTag> keyFeatureTagMap = suggestionConfig.getKeyFeatureTagMap(wrapperId);
            if (keyFeatureTagMap != null && !keyFeatureTagMap.isEmpty()) {
                // 加权：指定排序规则加权、原始权重
                for (SuggestSolrBean solrBean : suggestSolrBeanList) {
                    FeatureTag tag = keyFeatureTagMap.get(Integer.parseInt(solrBean.getValue()));
                    if (tag != null) {
                        solrBean.addScore(ScoreRankTypeEnum.POINTED_SORT_SCORE,
                                FeatureTagTypeEnum.getCustomScoreByType(tag.getType()), 1.0);
                    }
                    solrBean.addScore(ScoreRankTypeEnum.ORIGIN_WEIGHT, solrBean.getOriWeight(), 0.3);
                }
            } else {
                // 加权：原始权重
                suggestSolrBeanList.forEach(
                        solrBean -> solrBean.addScore(ScoreRankTypeEnum.ORIGIN_WEIGHT, solrBean.getOriWeight(), 0.3));
            }
```



####  FilterRecallStrategy

>  二框用户输入含有特色且有多个特色且不只有特色 
>
> 复合筛选项（特色）召回场景实现类
>
> 格式：【地标+特色1+特色2】

```java

```

 [suggest.pdf](C:\Users\lelec_1.TUJIA\Desktop\suggest.pdf) 

<img src="D:\Typora\HelpJava\file.assets\1595574360236.png" alt="1595574360236" style="zoom: 200%;" />



##### 组装地标相关的bean

```java
private void buildWithRelateCondition(SuggestSolrBean solrBean, List<FeatureTag> featureTagList, List<String> tagList,
                                          SuggestionContext context) {
        String originName = solrBean.getName();
        int originSecondConditionCode = solrBean.getSecondConditionType().getCode();
        String originValue = solrBean.getValue();
        String historyShowName = "";
        // C国内：1人-16人+；一室-八室及以上
        boolean isNotCtripMainland = !(
                StringUtils.equalsIgnoreCase(context.getOriginPara().getRebuildWrapperId(), EnumWrapperId.WAP_CTRIPBNB.getDesc())
                        && context.getQueryBean().getInternalOverseas() == InternalOverseasEnum.Internal);
        for (String filter : tagList) {
            if (isNotCtripMainland) {
                // 除C国内以外的四居及以上、10人+，需要特殊处理
                if (StringUtils.equalsIgnoreCase(filter, String.valueOf(EnumSearchHouseModel.Four.getDesc()))) {
                    filter += StaticConstants.FOUR_RESIDENCE_SUFFIX;
                } else if (StringUtils.equalsIgnoreCase(filter, String.valueOf(EnumOccupantType.Ten.getDesc()))) {
                    filter += StaticConstants.TEN_PEOPLE_SUFFIX;
                }
            }

            historyShowName += (StaticConstants.PLUS + filter);
        }
        //城市+特色+特色
        historyShowName = originName + historyShowName;

        String landmarkConditionValue;
        if (solrBean.getSecondConditionType() == SecondConditionTypeEnum.LANDMARK) {
            landmarkConditionValue = originSecondConditionCode + StaticConstants.UNDERLINE + solrBean.getLongitude()
                    + StaticConstants.COMMA + solrBean.getLatitude();
        } else {
            landmarkConditionValue = originSecondConditionCode + StaticConstants.UNDERLINE + originValue;
        }

        // 添加特色
        solrBean.setStdName(historyShowName);
        solrBean.setName(historyShowName);
        solrBean.setShowName(historyShowName);
        solrBean.setCityId(solrBean.getCityId());
        solrBean.setCity(solrBean.getCity());
        solrBean.setValue(String.valueOf(featureTagList.get(0).getFeatureTagValue()));
        solrBean.setConditionType(ConditionTypeEnum.FILTER);
        // SecondConditionType置为UNDEFINED
        solrBean.setSecondConditionType(SecondConditionTypeEnum.UNDEFINED);
        solrBean.setHistoryShowName(historyShowName);

        List<FilterCombineCondition> singleCondition = Lists.newArrayList();
        // 地标
        FilterCombineCondition relateCondition = FilterCombineCondition.builder().conditionDesc(originName)
                .enumSuggestionConditionType(ConditionTypeEnum.LOCATION.getCode())
                .suggestionConditionValue(landmarkConditionValue).build();
        singleCondition.add(relateCondition);
        // 添加其余特色
        for (int i = 1; i < featureTagList.size(); i++) {
            FilterCombineCondition relateConditionRest = FilterCombineCondition.builder()
                    .conditionDesc(featureTagList.get(i).getFeatureTagName())
                    .enumSuggestionConditionType(ConditionTypeEnum.FILTER.getCode())
                    .suggestionConditionValue(String.valueOf(featureTagList.get(i).getFeatureTagValue())).build();
            singleCondition.add(relateConditionRest);
        }
        solrBean.setCombineConditions(singleCondition);
        // 命中特色场景得分
        solrBean.addScore(ScoreRankTypeEnum.HIT_FILTER_SCORE, 1.0, 0.5);
//        solrBean.addScore(ScoreRankTypeEnum.ORIGIN_WEIGHT, 1.0, 0.3);
    }
}
```

#### PositionTwoRecallStrategy

同defaultRankComponent

### 数据聚合

> 默认聚合组件
>
> http://wiki.corp.tujia.com/confluence/pages/viewpage.action?pageId=16584239

```java
class DefaultDataConvergeComponent implements DataConvergeComponent{}
```