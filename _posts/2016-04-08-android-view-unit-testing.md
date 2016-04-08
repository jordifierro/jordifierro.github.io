---
layout: post
title:  "Android view unit testing recipe"
date:   2016-04-08 12:00:00 +0100
categories: projects
comments: true
---
Developing the
[android-base](https://github.com/jordifierro/android-base)
project, I've learnt how to cook
delicious unit tests, specially designed for the Android view layer.
Today I'm going to explain how I prepare them.

With this recipe, you'll be able to test fragments and activities behavior,
making them play like puppets.
It can seem complicated to cook
and the tests will continue being Android dependent,
but the results are very grateful...
So I strongly recommend you to try it at home!

### Basic Ingredients

* [Model-view-presenter](https://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93presenter)
* [Dagger 2](https://guides.codepath.com/android/Dependency-Injection-with-Dagger-2)
* [Espresso](https://google.github.io/android-testing-support-library/docs/espresso/)
* [Mockito](http://mockito.org/)


### Method

#### 1. Make an app that's easy to test

The basic ingredient of this recipe is an app to be tested, of course.
But a simple app doesn't serve, it must be an app made to be tested.
It needs to be implementated following the MVP pattern
mixed with the dependency inversion principle
(I don't mean it's the only way to make an app easy to test,
I just list the patterns that we'll need here).

![Shadow puppets](/assets/images/shadow_puppets.jpg)

The views must be passive and free of any logic,
moved like puppets by the corresponding presenter.
This one is in charge of receiving the view events and send orders to it.
The dependency injection with Dagger makes the component replacement easier.
Thus, on testing we will be able to mock the presenter and
inject it to the view, send our own orders to it and check the callbacks.

We will then be able to test a more concrete chain of events.
Furthermore, we isolate the rest of the app,
focusing just on possible view related errors.

*__Note:__ Actually, 'views are completely fool' is a little white lie.
Due to Android devices fragmentation, the views are structured in fragments,
whom are included in activities depending on the size of the screen.
These activities change, for example, if the app is for mobile or tablet,
and its navigation too.
For this reason, navigation must be placed inside the activities
(but just navigation, the rest of the logic must be outside the views).*

#### 2. Gradle the ingredients together

Add the [android-apt](https://bitbucket.org/hvisser/android-apt) library,
necessary to compile Dagger files, to the project root `build.gradle`:

{% highlight java %}
...
buildscript {
    repositories {
        jcenter()
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:2.1.0-alpha4'
        classpath 'com.neenbedankt.gradle.plugins:android-apt:1.8'
    }
}
...
{% endhighlight %}

Then add all the necessary libraries to your project specific `build.gradle`
and define the `testInstrumentationRunner`:

{% highlight java %}
apply plugin: 'com.android.application'
apply plugin: 'com.neenbedankt.android-apt'
...
android {
    ...
    defaultConfig {
        ...
        testInstrumentationRunner 'android.support.test.runner.AndroidJUnitRunner'
    }
    ...
}
...
dependencies {
    ...
    androidTestApt      'com.google.dagger:dagger-compiler:2.0.2'

    androidTestCompile  'junit:junit:4.10'
    androidTestCompile  'org.mockito:mockito-core:1.9.5'
    androidTestCompile  'com.crittercism.dexmaker:dexmaker:1.4'
    androidTestCompile  'com.crittercism.dexmaker:dexmaker-dx:1.4'
    androidTestCompile  'com.crittercism.dexmaker:dexmaker-parent:1.4'
    androidTestCompile  'com.crittercism.dexmaker:dexmaker-mockito:1.4'
    androidTestCompile  'com.android.support:support-annotations:23.2.0'
    androidTestCompile  'com.android.support.test.espresso:espresso-core:2.2.1'
    androidTestCompile  'com.android.support.test.espresso:espresso-intents:2.2.1'
}
{% endhighlight %}

Remember to apply the `android-apt` plugin and add the `dagger-compiler`
with `androidTestApt` instead of `androidTestCompile`.

__Libraries summary:__

* [JUnit](http://junit.org/junit4/) is the framework to structure the tests.
* [Mockito](http://mockito.org/) will let us mock objects to
define their behavior and know when they're called.
* [DexMaker](https://github.com/crittercism/dexmaker)
is needed to run Mockito on Instrumentation tests.
* [AndroidSupportAnnotations](http://tools.android.com/tech-docs/support-annotations)
is defined here just to avoid a version dependency
problem (because it's included in other libraries).
* [Espresso](https://google.github.io/android-testing-support-library/docs/espresso/)
is the framework to write Android UI tests
(espresso-intents are added to test navigation that, as I've said, is
responsability of the activities).

You need to understand the role of each library
to follow this tutorial.

*__Note:__ Mockito cannot be updated to version 2
because of some `dexmaker` issues.*

#### 3. Prepare your first test (not working yet)

You can do it to your taste.
I usually launch the activity and get the target test fragment at the setup.
Then, I get its presenter and replace it by a mocked one.
Here is a test example for a two text field model edition view:

{% highlight java %}
@RunWith(AndroidJUnit4.class)
@LargeTest
public class EditViewTest {

    @Rule
    public final ActivityTestRule<EditActivity> activityTestRule =
                        new ActivityTestRule<>(EditActivity.class, true, false);
    private EditFragment editFragment;

    @Before
    public void setUp() throws Exception {
        this.activityTestRule.launchActivity(new Intent().putExtra(EditActivity.PARAM_ID, 2));
        this.editFragment = ((EditFragment) this.activityTestRule.getActivity()
                                .getFragmentManager().findFragmentById(R.id.fragment_container));
        this.editFragment.editPresenter = Mockito.mock(EditPresenter.class);
    }

    @Test
    public void testGetItemId() {
        assertEquals(2, this.editFragment.getItemId());
    }

    @Test
    public void testShowItem() {

        this.activityTestRule.getActivity().runOnUiThread(new Runnable() {
            @Override
            public void run() {
                EditViewTest.this.editFragment.showItem(
                        new ItemEntity("Title", "Content..."));
            }
        });

        onView(withText("Title")).check(matches(isDisplayed()));
        onView(withText("Content...")).check(matches(isDisplayed()));
    }

    @Test
    public void testEditItem() {
        this.activityTestRule.getActivity().runOnUiThread(new Runnable() {
            @Override
            public void run() {
                EditViewTest.this.editFragment.showItem(
                        new ItemEntity("Title", "Content"));
            }
        });

        onView(withId(R.id.et_title)).perform(typeText(" updated!"), closeSoftKeyboard());
        onView(withId(R.id.et_content)).perform(typeText(" changed!"));
        onView(withId(R.id.btn_submit)).perform(click());

        verify(this.editFragment.editPresenter).updateItem("Title updated!", "Content changed!");
    }

}
{% endhighlight %}

This approach seems to work (and it actually does it most of the times)
but it has 2 critical and unacceptable errors:

* You cannot know what has happened with
the presenter before the mock replacement.
* Real presenter is initialized and, consequently, all the other components
(data repositories, interactors...) when this screen is opened...
and that's not unit testing anymore.

So don't taste it yet, let's keep cooking!

#### 4. Make the DaggerTestComponent

We need to take action before the presenter injection is done.
That's the reason why we have to replace the Dagger `Component` and, to do it,
first we must create a test mocker `Component`.

The plan is to send that test `Component` to inject mocked presenters
to the fragments without the knowledge of them.
Create an interface and move there
all the methods from the old dagger `Component`.

{% highlight java %}
public interface FragmentInjector {

    void inject(EditFragment editFragment);

}
{% endhighlight %}

Then, inherit the new interface from the old real dagger component:

{% highlight java %}
@ActivityScope
@Component(dependencies = ApplicationComponent.class)
public interface ActivityComponent extends FragmentInjector {}
{% endhighlight %}

And from the new mocker component too:

{% highlight java %}
@ActivityScope
@Component(modules = TestMockerModule.class, dependencies = ApplicationComponent.class)
public interface TestMockerComponent extends FragmentInjector {}
{% endhighlight %}

Finally, implement the module for the test component to return a mocked presenter
when required.

{% highlight java %}
@Module
public class TestMockerModule {

    @Provides @ActivityScope
    EditPresenter provideEditPresenter() {
        return Mockito.mock(EditPresenter.class);
    }

}
{% endhighlight %}

Now, we must use the `FragmentInjector` type in the fragments, being it
an `ActivityComponent` or a `TestMockerComponent` depending on the case
(make the necessary code adjustments to achieve that).

#### 5. Add a test Application

To deal the mocker injector component to the fragment
(who executes the self-injection)
we must move the responsability of the component creation
to the `Application` class and replace it by another one
that will return the mocker component.

In my project I have a Dagger Component for the activity scope, and I used
to generate it in the `Activity`
(asking the `Application` for the `ApplicationComponent`).
With this structure we must move the creation of the `ActivityComponent` to
the `Application` class too, but we can keep the scope by saving the instance on
the activity as a singleton
(the `Application` just create a new one on each call).

{% highlight java %}
public abstract class BaseActivity extends AppCompatActivity {

    private FragmentInjector fragmentInjector;

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        this.initializeActivityComponent();
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_layout);
        ButterKnife.bind(this);
        this.initializeActivity(savedInstanceState);
    }

    private void initializeActivityComponent() {
        if (this.fragmentInjector == null) {
            this.fragmentInjector = ((BaseApplication)getApplication()).getFragmentInjector();
        }
    }

    public FragmentInjector getFragmentInjector() {
        return this.fragmentInjector;
    }

    ...

}
{% endhighlight %}

*__Note:__ The `ActivityComponent` is initialized before `onCreate` call
to avoid fragment's `onCreate` being called first
(
[like in this case](http://stackoverflow.com/questions/14093438/after-the-rotate-oncreate-fragment-is-called-before-oncreate-fragmentactivi)
), that would cause an attempt to fragment injection with a null injector.*

Once that is done, we must structure the `Application` class in such way that
we'll be able to override just one method to replace the mocker component.

`BaseApplication` class:

{% highlight java %}
public class BaseApplication extends Application {

    protected ApplicationComponent applicationComponent;

    @Override
    public void onCreate() {
        super.onCreate();
        this.initializeInjector();
    }

    protected void initializeInjector() {
        this.applicationComponent = DaggerApplicationComponent.builder()
                                        .applicationModule(new ApplicationModule(this))
                                        .build();
    }

    public ApplicationComponent getApplicationComponent() {
        return this.applicationComponent;
    }

    public FragmentInjector getFragmentInjector() {
        return DaggerActivityComponent.builder()
                .applicationComponent(this.applicationComponent).build();
    }

}
{% endhighlight %}

`TestMockerApplication` class:

{% highlight java %}
public class TestMockerApplication extends BaseApplication {

    @Override
    public FragmentInjector getFragmentInjector() {
        return DaggerTestMockerComponent.builder()
                .applicationComponent(this.applicationComponent)
                .testMockerModule(new TestMockerModule()).build();
    }

}
{% endhighlight %}

*__Note:__ In this step we can appreciate the scope of each Dagger component,
more specifically `ApplicationComponent` and `ActivityComponent`.
You must try to adapt it to your own project needs, always keeping
the instances saved in their corresponding place to avoid modifying their scope.*

#### 6. Insert the secret ingredient

To replace the `BaseApplication` class for the test one we need to create
a test runner, where we'll be able to select the `Application` class to be used.
Inherit from `AndroidJUnitRunner`, override the method `newApplication`
and place there the name of your `TestMockerApplication`.
{% highlight java %}
public class TestMockerRunner extends AndroidJUnitRunner {

    @Override
    public Application newApplication(ClassLoader cl, String className, Context context)
            throws InstantiationException, IllegalAccessException, ClassNotFoundException {
        return super.newApplication(cl, TestMockerApplication.class.getName(), context);
    }

}
{% endhighlight %}

Change the instrumentation runner in your project `build.gradle`,
replacing the package name:

{% highlight java %}
apply plugin: 'com.android.application'
apply plugin: 'com.neenbedankt.android-apt'
...
android {
    ...
    defaultConfig {
        ...
        testInstrumentationRunner 'com.your.package.TestMockerRunner'
    }
    ...
}
...
{% endhighlight %}

At last, the mocked presenter is injected automatically to the fragment,
so you can remove its replacement and verify their initialization calls.
You can now fix the previous test example:

{% highlight java %}
@RunWith(AndroidJUnit4.class)
@SmallTest
public class EditViewTest {

    @Rule
    public final ActivityTestRule<EditActivity> activityTestRule =
                        new ActivityTestRule<>(EditActivity.class, true, false);
    private EditFragment editFragment;

    @Before
    public void setUp() throws Exception {
        this.activityTestRule.launchActivity(new Intent().putExtra(EditActivity.PARAM_ID, 2));
        this.editFragment = ((EditFragment) this.activityTestRule.getActivity()
                                .getFragmentManager().findFragmentById(R.id.fragment_container));
    }

    @Test
    public void testInitPresenter() {
        verify(this.editFragment.editPresenter).initWithView(this.editFragment);
    }

    @Test
    public void testGetItemId() {
        assertEquals(2, this.editFragment.getItemId());
    }

    @Test
    public void testShowItem() {

        this.activityTestRule.getActivity().runOnUiThread(new Runnable() {
            @Override
            public void run() {
                EditViewTest.this.editFragment.showItem(
                        new ItemEntity("Title", "Content..."));
            }
        });

        onView(withText("Title")).check(matches(isDisplayed()));
        onView(withText("Content...")).check(matches(isDisplayed()));
    }

    @Test
    public void testEditItem() {

        this.activityTestRule.getActivity().runOnUiThread(new Runnable() {
            @Override
            public void run() {
                EditViewTest.this.editFragment.showItem(
                        new ItemEntity("Title", "Content"));
            }
        });

        onView(withId(R.id.et_title)).perform(typeText(" updated!"), closeSoftKeyboard());
        onView(withId(R.id.et_content)).perform(typeText(" changed!"));
        onView(withId(R.id.btn_submit)).perform(click());

        verify(this.editFragment.editPresenter).updateItem("Title updated!", "Content changed!");
    }

}
{% endhighlight %}

Also notice that now the test class is marked as `@SmallTest`.
That's because the rest of the app classes
(that includes database, network, threating... classes)
won't be used by the mocked presenter
(info about
[test sizes](http://googletesting.blogspot.com.es/2010/12/test-sizes.html))
because it works as a firewall.

And that's it! Enjoy your meal!

### Conclusions

With these method implemented, view testing becomes easy peasy.
Be creative with the possibilities of this kind of testing, and enjoy it!

Also, you can modify a little bit this recipe and create
integration Instrumented tests with a fake api
(using MockWebServer or creating a custom one)
or other kind of custom tests...

I hope this can be of any help. Don't hesitate to ask or make suggestions!

__Used resources:__

* [Source code](https://github.com/jordifierro/android-base)
* [Custom test runner](http://blog.sqisland.com/2015/12/mock-application-in-espresso.html)
* [MVP + Dependency Inversion project](https://github.com/android10/Android-CleanArchitecture)
* [Dagger 2 guide](https://guides.codepath.com/android/Dependency-Injection-with-Dagger-2)
