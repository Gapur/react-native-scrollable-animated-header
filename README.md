<p align="center">
  <img width="300"src="https://github.com/Gapur/react-native-scrollable-animated-header/blob/master/assets/react-native-example.gif">
</p>

# React Native ScrollView Animated Header

React Native Animated Header App with ScrollView

Animated header is the most common design pattern in today’s apps. Animations are an important part of mobile applications.

## Getting Started

1. Clone this repository
```
git clone https://github.com/Gapur/react-native-scrollview-animated-header.git
```
2. Install dependencies
```
yarn
```
3. Launch app
```
npm run ios # for npm
```

## Making Magic Code

We need to define some constants for the animated header which will be used to interpolate the scroll position value.

```js
const HEADER_MAX_HEIGHT = 240;
const HEADER_MIN_HEIGHT = 84;
const HEADER_SCROLL_DISTANCE = HEADER_MAX_HEIGHT - HEADER_MIN_HEIGHT;
```

I will display user data with ScrollView component. So We should to change App.js file like this:

```js
const HEADER_MAX_HEIGHT = 240; // max header height
const HEADER_MIN_HEIGHT = 84; // min header height
const HEADER_SCROLL_DISTANCE = HEADER_MAX_HEIGHT - HEADER_MIN_HEIGHT; // header scrolling value

// create array by 10 fake user data
const DATA = Array(10)
  .fill(null)
  .map((_, idx) => ({
    id: idx,
    avatar: faker.image.avatar(),
    fullName: `${faker.name.firstName()} ${faker.name.lastName()}`,
  }));

function App() {
  const renderListItem = (item) => (
    <View key={item.id} style={styles.card}>
      <Image style={styles.avatar} source={{uri: item.avatar}} />
      <Text style={styles.fullNameText}>{item.fullName}</Text>
    </View>
  );

  return (
    <SafeAreaView style={styles.saveArea}>
      <ScrollView contentContainerStyle={{ paddingTop: HEADER_MAX_HEIGHT - 32 }}> // it should be under the header 
        {DATA.map(renderListItem)}
      </ScrollView>
    </SafeAreaView>
  );
}
```

We need to create the header under the ScrollView. We will use Animated.View

```js
<Animated.View
  style={[
    styles.topBar,
    {
      transform: [{scale: titleScale}, {translateY: titleTranslateY}],
    },
  ]}>
  <Text style={styles.title}>Management</Text>
</Animated.View>
```

## Magic Animation

React Native provides Animated API for animations. Animated API focuses on declarative relationships between inputs and outputs, with configurable transforms in between, and start/stop methods to control time-based animation execution.

We are going to use Animated.ScrollView to make the scroll view, and attach a callback to listen to the onScroll event when it is changed. Then, using interpolation to map value between the y-axis and opacity. The interpolation maps input ranges to output ranges, typically using a linear interpolation but also supports easing functions.

Let’s update our App.js file with the following lines of code:

```js
const scrollY = useRef(new Animated.Value(0)).current; // our animated value

// our header y-axis animated from 0 to HEADER_SCROLL_DISTANCE,
// we move our element for -HEADER_SCROLL_DISTANCE at the same time.
const headerTranslateY = scrollY.interpolate({
  inputRange: [0, HEADER_SCROLL_DISTANCE],
  outputRange: [0, -HEADER_SCROLL_DISTANCE],
  extrapolate: 'clamp',
});

// our opacity animated from 0 to 1 and our opacity will be 0
const imageOpacity = scrollY.interpolate({
  inputRange: [0, HEADER_SCROLL_DISTANCE / 2, HEADER_SCROLL_DISTANCE],
  outputRange: [1, 1, 0],
  extrapolate: 'clamp',
});
const imageTranslateY = scrollY.interpolate({
  inputRange: [0, HEADER_SCROLL_DISTANCE],
  outputRange: [0, 100],
  extrapolate: 'clamp',
});

// change header title size from 1 to 0.9
const titleScale = scrollY.interpolate({
  inputRange: [0, HEADER_SCROLL_DISTANCE / 2, HEADER_SCROLL_DISTANCE],
  outputRange: [1, 1, 0.9],
  extrapolate: 'clamp',
});
// change header title y-axis
const titleTranslateY = scrollY.interpolate({
  inputRange: [0, HEADER_SCROLL_DISTANCE / 2, HEADER_SCROLL_DISTANCE],
  outputRange: [0, 0, -8],
  extrapolate: 'clamp',
});
```
Above, we use useRef to persist Animated value. useRef returns a mutable ref object whose .current property is initialized to the passed argument.

Next, We need to update Animated value when we are scrolling ScrollView. We will catch event by Animated.event. It maps directly to animated value scrollY and update it when we are panning or scrolling.

We can use the native driver by specifying useNativeDriver: true in our animation configuration.

Let’s animate our components by the following codes:

```js
<SafeAreaView style={styles.saveArea}>
  <Animated.ScrollView
    contentContainerStyle={{ paddingTop: HEADER_MAX_HEIGHT - 32 }}
    scrollEventThrottle={16} // 
    onScroll={Animated.event(
      [{ nativeEvent: { contentOffset: { y: scrollY } } }], // event.nativeEvent.contentOffset.x to scrollX
      { useNativeDriver: true }, // use native driver for animation
    )}>
    {DATA.map(renderListItem)}
  </Animated.ScrollView>
  <Animated.View
    style={[styles.header, { transform: [{ translateY: headerTranslateY }] }]}>
    <Animated.Image
      style={[
        styles.headerBackground,
        {
          opacity: imageOpacity,
          transform: [{ translateY: imageTranslateY }],
        },
      ]}
      source={require('./assets/management.jpg')}
    />
  </Animated.View>
  <Animated.View
    style={[
      styles.topBar,
      {
        transform: [{ scale: titleScale }, { translateY: titleTranslateY }],
      },
    ]}>
    <Text style={styles.title}>Management</Text>
  </Animated.View>
</SafeAreaView>
```

## Article on Medium

[React Native Scrollable Animated Header](https://medium.com/javascript-in-plain-english/react-native-scrollable-animated-header-6dfe453d7078)

## How to contribute?

1. Fork this repo
2. Clone your fork
3. Code 🤓
4. Test your changes
5. Submit a PR!
