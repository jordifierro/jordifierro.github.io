---
layout: post
title:  "Android Http Interceptor testing recipe"
date:   2016-04-13 10:00:00 +0100
categories: development
comments: true
---

Here it comes another recipe I've learnt during the
[android-base](https://github.com/jordifierro/android-base) development!
Today's cooking is really short and easy:
how to test your `okhttp3.Interceptor`
you use to add headers to your http requests.

![Antenna](/assets/images/antenna.jpg)

### Basic Ingredients

* [Retrofit](http://square.github.io/retrofit/)
/[OkHttp](http://square.github.io/okhttp/)
* [Interceptor](https://github.com/square/okhttp/wiki/Interceptors)
* [MockWebServer](https://github.com/square/okhttp/tree/master/mockwebserver)

### Method

#### 1. Create your Android client app

The basic ingredient is an Android client application that retrieves data
from an api server.
Usually, we modify the headers of the http requests made to that api.
It's a really common scenario, as common as not testing the headers sent.
Let's fix that!

In this particular case I use a
[Square](http://square.github.io/)'s client http library: Retrofit.
But it's also ok if you just use OkHttp,
because it's the really required library for this recipe
(Retrofit depends on it), or even another http library you prefer
(you'll just need to make some adaptations).

#### 2. Add an HttpInterceptor

The main ingredient of this recipe is the `Interceptor`.
This class is added to the `OkHttpClient` and it...

*...observes, modifies, and potentially short-circuits requests going out and
the corresponding requests coming back in.
Typically interceptors will be used to add, remove, or transform headers
on the request or response.*

I use it in [android-base](https://github.com/jordifierro/android-base)
to add two headers to all my api calls.
Setup the interception system is very easy.

You must already have the api client system developed.
Here just take a look at the `HttpInterceptor.java` example:

{% highlight java %}
@Singleton
public class HttpInterceptor implements Interceptor {

    @Inject
    public HttpInterceptor() {}

    @Override
    public Response intercept(Chain chain) throws IOException {
        Request request = chain.request().newBuilder()
                .addHeader("Accept-Language", Locale.getDefault().getLanguage())
                .addHeader("Accept", RestApi.VERSION_HEADER + RestApi.API_VERSION)
                .build();
        return chain.proceed(request);
    }

}
{% endhighlight %}

In this case, I send the device language via `Accept-Language` header
and also `application/vnd.railsapibase.v1` as the `Accept` header
to tell the server which api version and language the client wants to use.

Then, we need to add this interceptor to the `OkHttpClient`.
In my case I use Retrofit so I need to define its `OkHttpClient`
explicitly to add it.
I create it in a
[Dagger 2](https://guides.codepath.com/android/Dependency-Injection-with-Dagger-2)
module:

{% highlight java %}
...
    @Provides
    @Singleton
    RestApi provideRestApi() {
        OkHttpClient client = new OkHttpClient().newBuilder()
                                                .addInterceptor(new HttpInterceptor())
                                                .build();

        GsonConverterFactory factory = GsonConverterFactory.create(new GsonBuilder()
            .setFieldNamingPolicy(FieldNamingPolicy.LOWER_CASE_WITH_UNDERSCORES).create());

        return new Retrofit.Builder()
                           .baseUrl(RestApi.URL_BASE)
                           .addConverterFactory(factory)
                           .addCallAdapterFactory(RxJavaCallAdapterFactory.create())
                           .client(client)
                           .build()
                           .create(RestApi.class);
    }
...
{% endhighlight %}

Don't worry if you don't use Dagger, Retrofit or
[RxJava](https://github.com/ReactiveX/RxJava),
the only important part here is to `.addInterceptor(new HttpInterceptor())`
to your `OkHttpClient`.

Ok! By this time, you must be able to receive
the headers on your server requests.
Test it manually a little bit to be sure you have included correctly
the interceptor and let's keep cooking...

#### 3. Test it

As I've said, this recipe is really short.
We only need to add our key ingredient, the test, and that's it!

{% highlight java %}
public class HttpInterceptorTest {

    @Test
    public void testHttpInterceptor() throws Exception {
        MockWebServer mockWebServer = new MockWebServer();
        mockWebServer.start();
        mockWebServer.enqueue(new MockResponse());

        OkHttpClient okHttpClient = new OkHttpClient().newBuilder()
                .addInterceptor(new HttpInterceptor()).build();
        okHttpClient.newCall(new Request.Builder().url(mockWebServer.url("/")).build()).execute();

        RecordedRequest request = mockWebServer.takeRequest();
        assertEquals(Locale.getDefault().getLanguage(), request.getHeader("Accept-Language"));
        assertEquals(RestApi.VERSION_HEADER + RestApi.API_VERSION, request.getHeader("Accept"));

        mockWebServer.shutdown();
    }

}
{% endhighlight %}

*__Note:__ Remember to add
`testCompile 'com.squareup.okhttp3:mockwebserver:3.2.0'`
to your `build.gradle`.*

The test is composed by 4 parts:

* __Create the server:__ MockWebServer is an awesome library that let's you
mock a server and test your requests, responses, reproduce delays, errors...
Here we will use a tiny part of its potential, but it's enough for us.
Just create the `MockWebServer`, start it and enqueue a void response.

* __Create the client:__
`OkHttpClient` will execute the mock call to `MockWebServer`.
First we create the client, adding to it our test target `HttpInterceptor`.
Then execute a void call to `mockWebServer.url("/")`, just to send a request.

* __Check the request:__ Take the `RecordedRequest` from the server and
test that its headers have the values that the `Interceptor` put there.

* __Shutdown the server:__ Don't forget it!

That's all folks!

[(Source code)](https://github.com/jordifierro/android-base)
