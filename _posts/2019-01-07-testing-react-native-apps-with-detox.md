---
layout: post
title: Testing react native applications with Detox
date: 2019-01-07 22:54:18
summary: Explores the use of detox for the end to end testing of react native applications
comments: true
categories: programming
thumbnail: programming
tags:
 - react-native
 - detox
 - end to end testing
 - e2e
---

[React Native][1] is getting pretty popular amongst developers and more and more applications are now being built on the top of it.
One of the important things to consider while developing any mobile application is to have proper tests. While developers often tend
to focus more on unit and integration tests, the end to end tests are equally important and often get overlooked to save time. There are several
advantages to writing e2e tests:
- While unit tests check if the blocks of code function independently, these tests ensure that the entire flow of the
application works in real-world scenarios.
- We will also end up saving a lot of time doing manual testing if the tests are automated.

We at [Skyscanner][2] are using [Detox][3] for end to end testing our mobile applications. Detox is a gray box end-to-end testing and automation library
which has support for react native and pure native projects. Detox supports both Android and iOS. This is a small tutorial that explains how you can get Detox up and
running in no time for your own application. I will only focus on iOS in this tutorial but running the same tests on android requires very few changes. 

I have written a really simple react native application for this tutorial. The source code for it is available [here][4]. This application
just consists of one button that toggles the visibility of an image. The main component `App` in `App.js` file looks like this:
{% highlight javascript %}
{% raw %}
export default class App extends Component {
  constructor(props) {
    super(props);
    this.toggleVisibility = this.toggleVisibility.bind(this);
    this.state = {
      isSecretVisible: false
    };
  }

  toggleVisibility() {
    this.setState({
      isSecretVisible: !this.state.isSecretVisible
    });
  }

  render() {
    return (
      <View testID='mainScreen' style={styles.container}>
        <Button testID='secretButton' 
          style={styles.welcome} 
          onPress={this.toggleVisibility}
          title={'Press to reveal secret'}/>
        {this.state.isSecretVisible && (
          <Image
          testID='secretImage'
          style={{width: 50, height: 50}}
          source={{uri: 'https://facebook.github.io/react-native/docs/assets/favicon.png'}}
        />)}
      </View>
    );
  }
}
{% endraw %}
{% endhighlight %}

This application looks something like this:

![Demo app](/images/demo_app.gif)

You will see a special property called `testID` assigned to some of the views. This is how react native uniquely 
identifies the views. We will see this in action when we start writing our tests. Follow this [guide][5] to get 
detox up and running. Detox can basically run with any test runner of our choice. We will be using [jest][6] in 
this tutorial. 
{% highlight bash %}
npm install -g detox detox-cli
{% endhighlight %}

You can check if detox is correctly installed by using:
{% highlight bash %}
detox --help
{% endhighlight %}

If everything is fine, you should see the list of commands supported by detox. First, 
we need to add configuration required for detox in `package.json`. Sample configuration looks like this:
{% highlight javascript %}
"detox": {
  "configurations": {
      "ios.sim.debug": {
      "binaryPath": "ios/build/Build/Products/Debug-iphonesimulator/DetoxTesting.app",
      "build": "xcodebuild -project ios/DetoxTesting.xcodeproj -scheme DetoxTesting -configuration Debug -sdk iphonesimulator -derivedDataPath ios/build",
      "type": "ios.simulator",
      "name": "iPhone X"
     }
  }
}
{% endhighlight %}

The configuration basically tells detox how to build the application, and what simulator to use. Make sure you have the 
chosen simulator available before you run the tests. 

Now let's write a test. Run:
{% highlight bash %}
detox init -r jest
{% endhighlight %}

You will see a new directory in your project called `e2e` with some basic code to run the first test. Let's add our first 
test to the file `firstTest.spec.js`. We will test if the only button in our application indeed toggles the visibility 
of the image. 
{% highlight javascript %}
describe('Test Secret Button', () => {
  beforeEach(async () => {
    await device.reloadReactNative();
    await device.disableSynchronization();
  });

  async function tapButton() {
    await waitFor(element(by.id('secretButton'))).toBeVisible().withTimeout(5000);
    return element(by.id('secretButton')).tap()
  }

 it('should have welcome screen', async () => {
    await expect(element(by.id('mainScreen'))).toBeVisible();
  }); 

  it('should reveal secret after tap', async () => {
    await tapButton();
    await waitFor(element(by.id('secretImage'))).toBeVisible().withTimeout(5000);
    return expect(element(by.id('secretImage'))).toBeVisible();
  });

  it('should hide secret after two taps', async () => {
    await tapButton();
    await tapButton();
    await waitFor(element(by.id('secretImage'))).toBeVisible().withTimeout(5000);
    return expect(element(by.id('secretImage'))).toBeNotVisible();
  });
});
{% endhighlight %}

Let's see this bit by bit. Before each test, we reload the react native application using `device.reloadReactNative()`.
To reduce flakiness of the tests, Detox provides us with a special function that will synchronize the test with the app.
You can do this by using `device.disableSynchronization()`. 

Let's see our first test case:
{% highlight javascript %}
it('should have welcome screen', async () => {
  await expect(element(by.id('mainScreen'))).toBeVisible();
});
{% endhighlight %}

This just checks if a view with `testID` equal to `mainScreen` is visible when the app is launched. 

Our next test is a little more complicated but still easy to understand:
{% highlight javascript %}
it('should reveal secret after tap', async () => {
  await tapButton();
  await waitFor(element(by.id('secretImage'))).toBeVisible().withTimeout(5000);
  return expect(element(by.id('secretImage'))).toBeVisible();
});
{% endhighlight %}

We first try to find the button and then tap it. This is done in the `tapButton` method:
{% highlight javascript %}
async function tapButton() {
  await waitFor(element(by.id('secretButton'))).toBeVisible().withTimeout(5000);
  return element(by.id('secretButton')).tap()
}
{% endhighlight %}

We used the `withTimeout` function because UI tests are flaky and it takes undetermined time for the screen to render. 
If our tests run before the screen is rendered, they will fail. Once the button is tapped, we just check if our secret
image is visible. Our next test checks if tapping the button twice again hides the secret image. 

To run the tests, use:
{% highlight javascript %}
detox build
{% endhighlight %}

This will build the application file that can be installed in the simulator. Next, run:
{% highlight javascript %}
detox test
{% endhighlight %}

Usually, when running the test suite, the device runs in headless mode. 
But if you open Simulator.app in OSX, you should see the tests in action. 

![Demo tests](/images/demo.gif)

That's it. We have our first e2e tests ready.  

 [1]: https://facebook.github.io/react-native/
 [2]: https://www.skyscanner.net/
 [3]: https://github.com/wix/Detox
 [4]: https://github.com/nerandell/DetoxTestingSample 
 [5]: https://github.com/wix/Detox/blob/master/docs/Introduction.GettingStarted.md
 [6]: https://jestjs.io/
