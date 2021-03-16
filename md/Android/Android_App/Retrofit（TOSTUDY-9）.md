Retrofit（TOSTUDY-9）

Retrofit（TOSTUDY-9）

Retrofit

1、加载依赖，在build.gradle中添加

    implementation 'com.squareup.retrofit2:retrofit:2.1.0'
    implementation 'com.squareup.okhttp3:okhttp:3.9.1'
    implementation 'com.squareup.retrofit2:converter-gson:2.1.0'

2、初始化Retrofit

Retrofit retrofit = new Retrofit.Bruilder().baseUrl("http://127.0.0.1/").build();

其中url(http://127.0.0.1/)必须以“/”结尾，否则会包illegleArgument

3、定义调用接口
public interface A{
    @GET("/blog?key=value")
    Call<ResponseBody> getBlog()

    @GET("/blog/{id}")
    Call<ResponseBody> getBlogByID(@Path("id") int id)

    @HTTP(method="PUT", path=""  hasBody=true)
}

4、生成接口A的代理
A a = retrofit.create(A.class)

5、获取Call
Call<ResponseBody> call = a.getBlog()
Call<ResponseBody> call = a.getBlogByID()

6、执行网络请求
call.enqueue