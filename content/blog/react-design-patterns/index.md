---
title: React design patterns
date: "2019-04-06T23:46:37.121Z"
---

Design pattern - is a solution to recurring problems. I like to think of design patterns as a recipes that help us cook delicious react apps. I hope this blog post will add new recipes to your cook book.

A disclaimer - I didn't invent these recipes myself. I rather gathered them from the community.

3 recipes that I'm going to introduce here are:

- Context API
- Presentation/Container Components
- Render Props

All these patterns have to do with managing state. I think it was Kyle Simpson who said - 90% of bugs come from improper state management. If you truly understand the state in your app that means you will be able to have a fine-grained control over your app.

---

## Cookbook Recipe #1: Context API

This is a recipe championed by Kent C Dodds. At least I heard about it from him first. This API was part of React a long time ago, however a new version came out of it and that was the time when Kent reintroduced the idea of using the context API.

Passing props can become very annoying and frustrating once you are passing them more than 1 level deep. This is a problematic because we end up passing props through a bunch of intermediary components that don't need that information. It's also possible that through this journey during this time as it moves between all of these props sometimes it changes name, for some reason the developer decides that name needs a change and so later when you come back and you try and find the origin of this prop you have like four or five files open as you're trying to track it all the way back up the chain. When Kent C Dodds mentions about this problem he referredf to it as 'prop drilling'. For people who are new to react it's something that they experience quite early on in their journey. You would usually want to find if there's an easier way to resolve this. Probably the intuitive way of finding a solution would be using a redux library. But it comes with a lot of the boilerplate code. And if you are new to this concpet then it's quite hard to get a good understanding quickly. Sometimes we want our state to be global.

`Context API`: "a way to pass data through the component tree without having to pass props down manually at every level." - React Documentation. This API gives us the ability to access a prop from lower down our tree without needing to pass it through intermediary components. The way API accomplishes it is using two key components. The first one is called Provider - you can think of it as a global state container. It provides the data we need to the second key component - Consumer. So the consumer has access to the provider anywhere in the app.

First thing you need to do is to call a method that's built in to React `React.createContext()` and assign it to the destructuder Provider and Consumer components.

```jsx
const { Provider, Consumer } = React.createContext()
```

In the Provider component we want to pass a prop of a value. Whatever we pass there that's going to be accessible by the consumer.

```jsx
<Provider value={/* some value */}
```

Consumer is going to be rendered somewhere else in our component tree. It needs to be rendered in a component that is a child of the provider esentially.

```jsx
<Consumer>
  {value => /* render something based on the context value */}
</Consumer>
```

### Example

Let's say we have an App component that's storing a user data in it's state. We are going to take that data and pass it to the Provider component through a value prop. Within the Provider we render a Dashoard component.

```jsx
class App extends Component {
  state = {
    user: {
      name: "Aurimas",
    }
  }
  render() {
    <Provider value={this.state}>
      <Dashboard />
    <Provider>
  }
}
```

As we move on further down our component tree - we can see that the Dashboard component renders Sidebar component. Sidebar component doesn't take any props in it at all but as we render it we are also rendering Consumer component. Inside we have a function that receives a parameter `value` and here we can reach the `name` value - from the App components inner state, that was passed to the Provider.

```jsx
const Dashboard = () => {
  return <Sidebar />
}

const Sidebar = () => {
  return <Consumer>{value => <h1>{value.user.name}</h1>}</Consumer>
}
```

---

You can also have consumers at multiple levels of the component tree. So if the data needs to be accessed in multiple different areas of you app, you can just render additional consumers there and each one of them will have access to that value from the app component.

Huge benefit of this pattern is essentially we're eliminating this issue of prop drilling of needing to pass this prop down multiple levels inside of our app and we're also eliminating the need for Redux. Just want to mention that I'm not saying that you don't need redux in your app from now on, but all I'm saying is a lot of the common use cases that people initially use redux for which is to avoid this sort of prop drilling you can just start by using the context API so it gives you the benefit of deferring that decision to a later date down the road of your web applications life.

### When to use Context API?

- When you find yourself passing props down deep into your tree.
- You truly need 'global state' - auth, user settings, translations, themes.
- Someone tells you that you need redux.

### What can you cook with this recipe?

- It can be a shopping cart system, if you do some kind of shopping cart management in your app - context API is really good for keeping track of whether the cart is open or closed, or what's in its inventory.
- If you want to do internatiolization - to manage translations across the app and you need to know what language is currently active and where you want to store translation strings.
- It's helpful for modals - in different parts of your application you need to know how to be able to open/close modal from anywhere.
- General Theme Provider - if you want to have a specific styling that you want to manage and many different levels inside of an app

---

## Cookbook Recipe #2: Presentation / Container components

Imagine you need to build a weather widget component. In terms to build it - you probably need to think of 2 things at the begining - `request` data from some weather API and `display` that data on the page. Seems like a simple task, and you would probably start to go about it like so:

```jsx
class WeatherWidget extends React.Component {
  state = { weather: null }

  componentDidMount() {
    // fetch weather data
    getWeather().then(weather => {
      this.setState({ weather })
    })
  }

  render() {
    // display weather data
    return (
      <div>
        <h1>
          {weahter.city} - {weather.country}
        </h1>
        <ul>
          <li>{weather.temp}</li>
          <li>{weather.description}</li>
        </ul>
      </div>
    )
  }
}
```

And this is fine, you probably would think that your component here is really small, portable, etc.. but what happens over time - you need to do more stuff. You might need to add a 12 day forecast, be able to show Fahrenheit and Celsius and all kinds of other things about the weather. And then you start to expand your component and it ends up looking a little bit like 200-300 lines of code component. Essentially it would be a bloated component - just simply too big.

### Problem

- Component is difficult to reason about - you look at it and think: "wtf is going on here..?"
- It is hard to modify and reuse it - you try and change it and it starts to break and can't use in other places because it's so specific to the current situation
- Undfriendly to other developers - feels like no one can touch this component unless is ready to deal with consequences

### Solution

Let's introduce a `separation of concerns`. It is an incredibly powerful concept when it comes to web application architecture and just programming in general. What is a concern? Well, it is something your code is responsible for, something your code cares about.

If we think about the weather widget component it kind of cares about two things - fetching the data and displaying it. So these are the two concerns. The idea of fetching data is a logic based concer. It is responsible for managing state - requesting the data from the API and then displaying that data on the screen - that's a UI concern, it has to do with a visual representation of it. So we can take these two kind of disparate but connected things and we actually separate them into two different components.

First we have our container component. It will be responsible for managing the logic of our weather widget. In other words - what the component actually does. In weather widget case - the responsibility is to get the data from API and store it to the state. And we should call this component `<WeatherWidgetContainer>...</WeatherWidgetContainer>`

Then we have the presentation component - responsible for a user interface. It should answer the question - "What the component looks like".

What should happen naturally, is that the container component should render the widget component and pass in any data that the widget needs to know about.

```jsx
class WeatherWidgetContainer extends React.Component {
  state = { weather: null }

  componentDidMount() {
    getWeather().then(weather => {
      this.setState({ weather })
    })
  }

  render() {
    return <WeatherWidget weather={this.state.weather} />
  }
}
```

Then, the `WeatherWidget` component should look like this. As you can see the only concern for this component is to receive props and render them to the screen.

```jsx
const WeatherWidget = props => {
  const { weather } = props
  return (
    <div>
      <h1>
        {weather.city} - {weather.country}
      </h1>
      <ul>
        <li>{weather.temp}</li>
        <li>{weather.description}</li>
      </ul>
    </div>
  )
}
```

Couple things you can notice right away is that we can use functional component for displaying UI. There is no need for a class based component, here is no state to manage and there are no life cycle methods we need to take care of. What that means, that this component is much more simple and therefore much easier to reason about.

### Why this pattern is useful?

- Enhance readability - you can find bugs easier, because if there's an issue with the API, you know which component to go to and fix it. Same goes for the UI - if there's a glitch or some component doesn't display data correctly - you will know where to go.
- Reusability - technically you can reuse this component in other places of your app. Just one thing that I need to mention is - it will need to receive identical props as it would from the current container and that might be problematic.
- Testable - for presentational component we can do snapshot testing, mock the props that are being passed in and then you have a total coverage on that component. For the container component we can introduce unit test with more of a kind of logical based approach.

### When to use this recipe?

- When your component is doing too many things
- There is a mixture of behaviour and presentation
- You find your component bloated - it has hundreds and hundreds lines of code

---

## Cookbook Recipe #3: Render Props

This recipe is actually a part of React documentation.

### Why would you use it?

I want to illustrate it with an example. Let's imagine you need to build a voting app for adorable puppies. And the user can open the app and vote if a dog is adorable or incredibly adorable. So you start building your app and the first thing you build is a puppy feed component and call the API to get the puppies. You would start with the list of things you need to do:

1. GET /api/puppies
2. Surface 'loading' spinner
3. Render adorable puppies
4. If error, render error

You are super happy about the work you've done. Everything is well tested, users are happy.. but the manager comes to you and asks to do the same thing, but instead of puppies - do it with kittens. Well, 'no problem' you think. We just follow the similar pattern and do something like:

1. GET /api/`kittens`
2. Surface 'loading' spinner
3. Render adorable `kittens`
4. If error, render error

And you are thinking - 'interesting, it's like we're doing the same thing twice'. We are doing kind of the same thing, but in once case we render puppies and in another case - kittens. You put your DRY (do not repeat yourself) hat on, you get very excited because this is a perfect abstraction opportunity here. And you go about this by creating a `<Petfeed />` component. The way it is going to work, is by passing different prop to this component.

```jsx
<Petfeed pet='kittens' />
<Petfeed pet='kittens' />
```

After you celebrate your perfect quality code - the next feature request comes in - we need to have a user profile component. The user profile is going to do a few things:

1. GET /api/`profile`
2. Surface 'loading' spinner
3. Render `user's profile`
4. If error, render error

We have a problem here. We cannot reuse the Petfeed component here simply because with the user profile we are not going to have a gallery of adorable pets. So how to use PetFeed component here? On top of that - while you are thinking about the implementation of this new feature, another request comes in - depending on if it's loading a puppies or kittens - we want to show different spinner. So now maybe you think you need a conditional statement in Petfeed component and you start to feel that this component will start to become a spaghetti code and not so reusable and extendible afterall. It's an abstraction that isn't working anymore. On top of that - we can't even use it for the user settings page either.

### Why did this happened?

Tight coupling of logic and presentation made our component less reusable and flexible. Because we connected the logic and the presentation together we kind of got backed into a corner prematurely.

### Solution

`Render props` - a simple technique for sharing code between React components. You have some logic and you have a bunch of components that wants to use that logic, so you need to use this render props to share it. It looks like this:

```jsx
<MyComponent render={data => <div>{data}</div>} />
```

The way we can use this, we need to realise what common things our components has:

- request data from API
- need loading states
- have error states

But they differ in a way that they render different things and need to render different spinners, different error states. We have all these different presentational concerns that we're not going to know about until they come back from the API. We want to deffer the decision-making around what they look like to the latest time possible. So they have common logic and different presentation.

The way we are going to resolve this issue is by renaming our `<Petfeed />` component to `<Resource />`.

When doing a render prop approach what we need to do in Resource component, in the `render()` method we call `this.props.render(this.state)` and we pass data from the state that we care about later. It can containt puppies, kittens or user settings. Essentially it's going to contain an API payload of data and we want to know wheter it's currently loading.

```jsx
class Resource extends Component {
  state = {
    loading: false,
    payload: [],
  }

  componentDidMount() {
    this.setState({ loading: true })
    axios.get(this.props.path).then(res => {
      this.setState({
        payload: res.data,
        loading: false,
      })
    })
  }

  render() {
    return this.props.render(this.state)
  }
}
```

The way we use this is strightforward. The `path` prop (the one we used in componentDidMount earlier) tells us what endpoint we pass render prop here, that has a payload with it. The payload contains the state of the resource component.

```jsx
<Resource
  path="/api/puppies"
  render={data => {
    if (data.loading) return <p>Loading Puppies...</p>
    return data.payload.map(puppy => <div>{puppy}</div>)
  }}
/>
```

We have access to all the props that was passed in. What it means that we can actually defer the decision-making around how these components render to the specific place that we want them to render.
The user profile page which is completely different than the other ones, all that logic around requesting, loading, etc, that can all be handled by the resource component and we can have completely different style of presentation

```jsx
<Resource
  path='/api/profilee'
  render={data => {
    if (data.loading) return <p>Loading Profile...</p>
    return (
      <div>
        <h1>{data.payload.name}</h1>
      </div>
    )
  }}
```

### Why render props are so important

- self document incredibly well
- separate presentation from logic
- decrease code duplication
- extrendible
- reusable
- abstract away logic

### when to use them

- Don't start with a render prop. Start simple and refactor later - if you start early you will get this wrong. The reason why you will get this wrong is because you don't know the use case yet.
- Share logic between components but not UI
- Other developers to re-use your code
- Defer rendering UI to a later date
- Dealing with common states/events in different components
