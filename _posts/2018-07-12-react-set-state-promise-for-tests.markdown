---
layout: post
title: "Providing a setState completion Promise for tests"
date: 2018-07-12 20:06:26
tags:
  - javascript
  - reactjs
  - testing
  - tdd
  - async
  - promise
  - enzyme
  - mocha
---

After upgrading our live system from Node 6 to Node 8 I got super excited about acquiring the
async/await capability, and was in that raving “async/await all the things!” mode. It was very
satisfying to see the callbacks-heavy production and test code to practically halve in size. Not
only was it less code now, but also the code structure got completely flat and trivial to follow.

[Some things are harder to async/await](https://github.com/facebook/react/issues/2642) than
others. Today I faced the following problem.

I have a tiny React component which is [a star rating
input.](http://ikr.su/h/react-star-rating-input/demo.html) It highlights the prospective star rating
value on mouse hover, and sets the new value on mouse click. The prospective value is stored in the
component's `state`, which is updated on `mouseEnter`/`mouseLeave` DOM event. Then the
`state.prospectiveValue` is used to highlight the potential rating selection.

I don't always write React components, but when I do, I test-drive them. Let me replay here my
`state.prospectiveValue` support implementation, driven by tests, [in
Mocha](https://mochajs.org/). First, we mount the component with [enzyme,](http://airbnb.io/enzyme/)
and fabricate some initial `prospectiveValue`:

{% highlight javascript %}
let wrapper

beforeEach(done => {
    wrapper = mount(<StarRatingInput value={1} />)
    wrapper.setState({ prospectiveValue: 2 }, done)
})
{% endhighlight %}

Mind the `done` callback passed to `setState`, to make sure that Mocha [waits for the state change
to succeed](https://reactjs.org/docs/react-component.html#setstate) before proceeding with each of
the test cases. Here's the first one of them:

{% highlight javascript %}
it('sets new prospective value on mouse hover for a star', () => {
    // Star at 0-based index 3 corresponds to a rating value of 4
    wrapper.find('a.star-rating-star').at(3).simulate('mouseEnter')
    assert.strictEqual(wrapper.state().prospectiveValue, 4)
})
{% endhighlight %}

The test is red: `2 != 4`. Let's make it green, changing the React component code.

{% highlight javascript %}
handleStarMouseEnter(value) {
    this.setState({ prospectiveValue: value })
}
{% endhighlight %}

Obviously.

Well, the test is still red. `setState` is asynchronous, and we have to pass it a callback, like we
did in `beforeEach()`, in order to wait for its conclusion. How can we possibly do that now? My
first thought was introducing a func prop `onSetStateDone`, and passing it to all the `setState()`
calls within the React component. Another option, which I learned about later, is [overriding the
componentDidUpdate.](https://github.com/facebook/react/issues/2642#issuecomment-66676469) Both ways
would require pyramids of callbacks to implement. Meh. Could that instead be a nail for my new shiny
async/await hammer? You bet!

{% highlight javascript %}
private _promiseSetStateDone: Promise<void>

get promiseSetStateDone() {
    return this._promiseSetStateDone
}

constructor(props: Props) {
    super(props)
    this._promiseSetStateDone = Promise.resolve()
    this.state = { prospectiveValue: 0 }
}

{% endhighlight %}

Then, instead of

{% highlight javascript %}
this.setState({ prospectiveValue: value })
{% endhighlight %}

we'll do

{% highlight javascript %}
this._promiseSetStateDone = new Promise(resolve => {
    this.setState({ prospectiveValue: value }, resolve)
})
{% endhighlight %}

Resulting in the test code like this:

{% highlight javascript %}
it('sets new prospective value on mouse hover for a star', async () => {
    // Star at 0-based index 3 corresponds to a rating value of 4
    wrapper.find('a.star-rating-star').at(3).simulate('mouseEnter')
    await wrapper.instance().promiseSetStateDone

    assert.strictEqual(wrapper.state().prospectiveValue, 4)
})
{% endhighlight %}

…And the test is now green. We can proceed test-driving.
