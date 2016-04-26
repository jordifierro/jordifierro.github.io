---
layout: post
title:  "Android restclient repository testing recipe"
date:   2016-04-26 22:00:00 +0100
categories: development
comments: true
---
And again... another Android testing recipe is here!!!

More specifically, the third one
(you can also check
[recipe \#1](/android-view-unit-testing),
[recipe \#2](/android-http-interceptor-testing),
and the
[rails-api-base](https://github.com/jordifierro/android-base)
project where all of them come from).

Today we are going to cook a test for data repository and restful client,
the piece of code in charge of making the api server calls.

![Phones](/assets/images/phones.jpg)

### Basic Ingredients

* [Retrofit](http://square.github.io/retrofit/)
* [RxJava](https://github.com/ReactiveX/RxJava)
* [MockWebServer](https://github.com/square/okhttp/tree/master/mockwebserver)

### Method

#### 1. Get a rest client

The basic ingredient of this recipe is a rest client to be tested.
This particular case is composed by a Repository and a RestApi.
The Repository is called when data is required and
it calls the RestApi client to retrieve the data from the server.
I've implemented my RestApi using
[Retrofit](http://square.github.io/retrofit/) with its
[RxJava](https://github.com/ReactiveX/RxJava) adapter.

Let's see an example of a single call through these elements.
Suppose we have a `Notes` model api with authentication.
A `note` is composed by a `title` and a `content` and
we need to authenticate ourself when managing notes.
In this particular case we want to create a note:

{% highlight ruby %}
INPUT:

POST /notes
Header['Authentication'] = params['auth_token']

    {
      "note" : {
        "title" : params["note"]["title"],
        "content" : params["note"]["content"]
      }
    }
_________________________________________________

OUTPUT:

status: 201 created
body:

    {
      "id" : int,
      "title" : "String",
      "content" : "String",
      "user_id" : int
    }
{% endhighlight %}

So the `RestApi.java` should look like this:

{% highlight java %}
public interface RestApi {

    @POST("/notes")
    Observable<Response<NoteEntity>> createNote(
        @Header("Authorization") String token, @Body NoteEntity note);

}
{% endhighlight %}

And the `NoteDataRepository.java`:

{% highlight java %}
@Singleton
public class NoteDataRepository extends RestApiRepository implements NoteRepository {

    private final RestApi restApi;

    @Inject
    public NoteDataRepository(RestApi restApi) {
        this.restApi = restApi;
    }

    @Override
    public Observable<NoteEntity> createNote(UserEntity user, final NoteEntity note) {
        return this.restApi.createNote(user.getAuthToken(), note)
                .map(new Func1<Response<NoteEntity>, NoteEntity>() {
                    @Override
                    public NoteEntity call(Response<NoteEntity> noteEntityResponse) {
                        handleResponseError(noteEntityResponse);
                        return noteEntityResponse.body();
                    }
                });
    }

}
{% endhighlight %}

_**Note:** In this case I use [Dagger 2](http://google.github.io/dagger/) to inject
the `RestApi`, but that's not essential._

`createNote` method just calls the `restApi` to create the note with its own
params and then maps the response event to unwrap the response.
`RestApiRepository` just implements the method `handleResponseError`
that checks if the `Response` is successful or throws an error otherwise.

(This is not necessary for this recipe, but you can check its source code
[here](https://github.com/jordifierro/android-base/blob/master/data/src/main/java/com/jordifierro/androidbase/data/repository/RestApiRepository.java)
).

You don't need to have exactly the same code or pieces as here,
this is only an example. Just be sure to have a Retrofit client using
RxJava adapter.

#### 2. Setup your gradle

The `build.gradle` should look like this:

{% highlight java %}
...
dependencies {
    ...

    compile     "io.reactivex:rxjava:1.1.1"
    compile     "com.squareup.retrofit2:retrofit:2.0.0-beta4"
    compile     "com.squareup.retrofit2:converter-gson:2.0.0-beta4"
    compile     "com.squareup.retrofit2:adapter-rxjava:2.0.0-beta4"

    testCompile "junit:junit:4.10"
    testCompile "com.squareup.okhttp3:mockwebserver:3.2.0"
}
...
{% endhighlight %}

[MockWebServer](https://github.com/square/okhttp/tree/master/mockwebserver)
is the secret ingredient we will use to test them all...

#### 3. Test it

First of all, we must setup the test: `NoteDataRepositoryTest.java`
(without tests yet):

{% highlight java %}
public class NoteDataRepositoryTest {

    private MockWebServer mockWebServer;
    private Gson gson;
    private NoteDataRepository noteDataRepository;
    private TestSubscriber testSubscriber;

    private UserEntity fakeUser;
    private NoteEntity fakeNote;

    @Before
    public void setUp() throws IOException {
        this.mockWebServer = new MockWebServer();
        this.mockWebServer.start();

        this.gson = new GsonBuilder()
                .setFieldNamingPolicy(FieldNamingPolicy.LOWER_CASE_WITH_UNDERSCORES).create();

        this.noteDataRepository = new NoteDataRepository(
                new Retrofit.Builder()
                        .baseUrl(mockWebServer.url("/"))
                        .addConverterFactory(GsonConverterFactory.create(this.gson))
                        .addCallAdapterFactory(RxJavaCallAdapterFactory.create())
                        .build()
                        .create(RestApi.class)
        );

        this.testSubscriber = new TestSubscriber();

        this.fakeUser = new UserEntity("some@mail");
        this.fakeUser.setAuthToken("AUTH_TOKEN");

        this.fakeNote = new NoteEntity("TITLE", "Content");
    }

    @After
    public void tearDown() throws Exception {
        this.mockWebServer.shutdown();
    }
    ...
{% endhighlight %}

Add them step by step:

* **MockWebServer:** To mock the responses and check the requests.
* **Gson:** To tell retrofit how to match the response json params
with java object attributes.
I use camelCase(client) to lowe_case_with_underscores(server).
_(you can skip this step)_
* **RestApi:** We create the retrofit class with our
RestApi interface, Gson converter, RxJava adapter and the url of the mockWebServer.
* **DataRepository:** Initialized with the previous RestApi, is our test target.
* **TestSubscriber:** RxJava test element to check the observable events.
* **UserEntity:** Used to send the token authentication param.
* **NoteEntity:** Used to mock the note params.

Don't forget to shut down the server after using it.

Everything is ready so... it's testing time!

{% highlight java %}
    ...
    @Test
    public void testCreateNoteRequest() throws Exception {
        this.mockWebServer.enqueue(new MockResponse());

        this.noteDataRepository.createNote(this.fakeUser, this.fakeNote)
                .subscribe(this.testSubscriber);

        RecordedRequest request = this.mockWebServer.takeRequest();
        assertEquals("/notes", request.getPath());
        assertEquals("POST", request.getMethod());
        assertEquals("AUTH_TOKEN", request.getHeader("Authorization"));
        assertEquals(this.gson.toJson(this.fakeNote).toString(),
                     request.getBody().readUtf8());
    }
    ...
{% endhighlight %}

This first test checks the output params, namely the request.
Realize that it just enqueues a simple `new MockResponse()`
because we don't care here about it (but we still need it).
After making the call to the repository, we get the `request`.

From that `request` we can check the path, method, headers, body...
The last sentence is the trickiest one,
but it just compares the json params sent with the
`fakeNote` object (we have set as param) json serialized.

And what about the response?

{% highlight java %}
    ...
    private static final String RESPONSE_BODY =
        '{"id":1,"title":"Title","content":"Content...","user_id":2}';

    @Test
    public void testCreateNoteSuccessResponse() throws Exception {
        this.mockWebServer.enqueue(
            new MockResponse().setResponseCode(201).setBody(RESPONSE_BODY));

        this.noteDataRepository.createNote(this.fakeUser, this.fakeNote)
                .subscribe(this.testSubscriber);
        this.testSubscriber.awaitTerminalEvent();

        NoteEntity responseNote = (NoteEntity) this.testSubscriber.getOnNextEvents().get(0);
        assertEquals(responseNote.getId(), 1);
        assertEquals(responseNote.getTitle(), "Title");
        assertEquals(responseNote.getContent(), "Content...");
        assertEquals(responseNote.getUserId(), 2);
    }
    ...
{% endhighlight %}

Here we add a response code and body to the MockResponse.
Then, we must subscribe the `testSubscriber` and make it wait.
`testSubscriber` gives us the response from the `createNote` call.
Realize that here we have double test:

* **First:** the json decode to our `NoteEntity` of the server response.
* **Second:** the map we apply inside the DataRepository
to the `Response<MessageEntity>` to convert it to `MessageEntity`.

### Conclusions

Although this is a simple example,
there are infinite possibilities and you can adapt it easily to your needs.
Give this recipe a try and enjoy testing your Android client application!

Comments and suggestions are welcome!

[(Source code)](https://github.com/jordifierro/android-base)
