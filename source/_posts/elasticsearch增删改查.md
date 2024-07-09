---
updated: 2024-2-18
created: 2024-2-18
title: elasticsearch增删改查
categories: Java
tags:
  - Java
excerpt: ''
cover: https://wallpaperm.cmcm.com/9cc01ff1758b776d7f5c2b91295b982b.jpg
---

#### yml配置

~~~yaml
spring:
  elasticsearch:
    urls: 62.234.46.178:9200
    usePassWord: false
~~~

#### 创建elasticesarch客户端

~~~java
@Configuration
@ConfigurationProperties(prefix = "spring.elasticsearch")
public class ElasticsearchConfig {
    
    private String username;
    private String password;
    private String urls;
    private String usePassWord;


    public void setUsername(String username) {
        this.username = username;
    }


    public void setPassword(String password) {
        this.password = password;
    }


    public void setUrls(String urls) {
        this.urls = urls;
    }


    public void setUsePassWord(String usePassWord) {
        this.usePassWord = usePassWord;
    }

    @Bean
    public RestClient restClient() {

        if (StringUtils.isEmpty(urls)) {
            throw new RuntimeException("elasticsearch.urls为空");
        }

        String trueStr = "true";

        String[] urlArr = urls.split(",");
        HttpHost[] httpHostArr = new HttpHost[urlArr.length];
        for (int i = 0; i < urlArr.length; i++) {
            String urlStr = urlArr[i];
            if (StringUtils.isBlank(urlStr)) {
                continue;
            }
            httpHostArr[i] = HttpHost.create(urlStr);
        }

        CredentialsProvider credentialsProvider = new BasicCredentialsProvider();
        if (trueStr.equals(usePassWord)) {
            //es账号密码（默认用户名为elastic）
            credentialsProvider.setCredentials(AuthScope.ANY, new UsernamePasswordCredentials(username, password));
        }

        ////创建请求头
        RestClientBuilder.HttpClientConfigCallback callback = httpAsyncClientBuilder -> httpAsyncClientBuilder
            .setDefaultCredentialsProvider(credentialsProvider)
            //  Todo 这里是关键  ######################解决 X-Elastic-Product  问题开始
            .setDefaultHeaders(
                    List.of(
                            new BasicHeader(
                                    HttpHeaders.CONTENT_TYPE, ContentType.APPLICATION_JSON.toString())))
            .addInterceptorLast(
                    (HttpResponseInterceptor)
                            (response, context) ->
                                    response.addHeader("X-Elastic-Product", "Elasticsearch"));
        // 创建低级客户端
        RestClient restClient = RestClient.builder(httpHostArr)
                .setHttpClientConfigCallback(callback).build();
        return restClient;
    }

    @Bean
    public ElasticsearchTransport elasticsearchTransport(RestClient restClient) {
    // 使用 Jackson 映射器创建传输
        ElasticsearchTransport elasticsearchTransport = new RestClientTransport(
                restClient, new JacksonJsonpMapper());
        return elasticsearchTransport;
    }


    @Bean
    public ElasticsearchClient elasticsearchClient(ElasticsearchTransport elasticsearchTransport) {
        // 并创建 API 客户端
        ElasticsearchClient elasticsearchClient = new ElasticsearchClient(elasticsearchTransport);
        return elasticsearchClient;
    }


}
~~~



#### elasticsearch7增删改查

~~~java
@RequestMapping("/createIndex")
    public Boolean createIndex() throws IOException {
        BooleanResponse exists = elasticsearchClient.indices().exists(ids -> ids.index(EsConstant.index));
        if(!exists.value()){
            //索引不存在
            CreateIndexResponse createIndexResponse = elasticsearchClient.indices().create(ids -> ids.index(EsConstant.index));
            return createIndexResponse.acknowledged();
        }
        return exists.value();
    }

    /**
     * 创建文档
     * @return
     */
    @RequestMapping("/createDocument")
    public IndexResponse createDocument() throws IOException {
        Student student = Student.builder()
                .id(UUID.randomUUID().toString())
                .userName("张三")
                .sex("男")
                .build();
        IndexResponse index = elasticsearchClient.index(ids ->
                ids
                        .index(EsConstant.index)
                        .id(student.getId())
                        .document(student));
        return index;
    }

    /**
     * 创建多个文档
     * @return
     */
    @RequestMapping("/createDocuments")
    public BulkResponse createDocuments() throws IOException {
        List<Student> students = Arrays.asList(new Student(UUID.randomUUID().toString(), "小红", "女"),
                new Student(UUID.randomUUID().toString(), "小白", "女"));
        return elasticsearchClient.bulk(ids->{
            ids.index(EsConstant.index);
            students.forEach(item->ids.operations(_0->_0.create(
                    _1->_1.id(
                            item.getId())
                            .document(item))));
            return ids;
        });
    }

    /**
     * 修改文档
     * @return
     */
    @RequestMapping("/updateDocument")
    public String updateDocument() throws IOException {
        Student student = Student.builder().id("6031433a-fca5-430e-a822-7d24c2e3467b")
                .userName("修改内容22")
                .sex("女").build();
//        elasticsearchClient.update(ids -> ids
//                .index(EsConstant.index)
//                .id(student.getId())
//                .doc(student), Student.class);
        UpdateRequest<Student, Student> request=UpdateRequest.of(ids->ids
                .index(EsConstant.index)
                .id(student.getId())
                .doc(student));
        elasticsearchClient.update(request,Student.class);
        return "修改成功";
    }

    /**
     * 删除文档
     */
    @RequestMapping("/delDocument")
    public String delDocument() throws IOException {
        String id = "6031433a-fca5-430e-a822-7d24c2e3467b";
//        elasticsearchClient.delete(ids->ids.index(EsConstant.index).id(id));
        DeleteRequest deleteRequest = DeleteRequest.of(ids->ids.index(EsConstant.index).id(id));
        elasticsearchClient.delete(deleteRequest);
        return "删除成功";
    }

    /**
     * 查询文档内容 精准匹配
     * @return
     */
    @RequestMapping("/queryDocument")
    public String queryDocument() throws IOException {
        List<Student> list = new ArrayList<>();
        SearchResponse<Student> search = elasticsearchClient.search(ids ->
                        ids.index(EsConstant.index)
                        .query(q -> q.term(te -> te.field("userName.keyword").value("小红")))
                , Student.class);
        List<Hit<Student>> hits = search.hits().hits();
        for(Hit<Student> hit:hits){
            list.add(hit.source());
        }
        return "结果集"+list;
    }
    /**
     * 查询文档内容 模糊匹配
     * @return
     */
    @RequestMapping("/queryDocumentWild")
    public String queryDocumentWild() throws IOException {
        List<Student> list = new ArrayList<>();
        SearchResponse<Student> search = elasticsearchClient.search(ids ->
                        ids.index(EsConstant.index)
                                .query(q->q.bool(boo->boo.must(mu->mu.wildcard(wid->wid.field("userName.keyword").value("小*")))))
                , Student.class);
        List<Hit<Student>> hits = search.hits().hits();
        for(Hit<Student> hit:hits){
            list.add(hit.source());
        }

        return "结果集"+list;
    }
    /**
     * 查询文档内容 builder形式
     * @return
     */
    @RequestMapping("/queryDocumentWildBuilder")
    public String queryDocumentWildBuilder() throws IOException {
        List<Student> studentList = new ArrayList<>();

        try {
            SearchResponse<Student> searchResponse = elasticsearchClient.search(ids->ids.index(EsConstant.index).query(query->{

                query.bool(boo->{
                    boo.must(mu->mu.wildcard(wid->wid.field("userName.keyword").value("张*")));
                    boo.must(mu->mu.term(ter->ter.field("sex").value("女")));
                    return boo;
                });

                return query;
            }), Student.class);
            List<Hit<Student>> hits = searchResponse.hits().hits();

            for (Hit<Student> hit : hits) {
                studentList.add(hit.source());
            }
        } catch (IOException e) {
            e.printStackTrace();
        }

        return "结果集"+studentList;
    }
~~~

