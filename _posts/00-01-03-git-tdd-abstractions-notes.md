---
layout: notes
title: Git, TDD & Abstractions Notes
date: 2014-05-14 00:00:00
---

## Class Flow

- Run through GitHub slides.
- Start GitHub example of simply updating the course members (see example
  below).
- Run through TDD slides.
- Do the TDD interest rate calculator.
- Have them do the TDD challenge.
- Continue with abstractions (_More Fun With Functions_).


## GitHub Example: Course Members

- Have students clone and make a pull request to add their name.
- Start on one side of the room and accept that pull request.
- Note that the pull request is able to be merged.
- Move to the next student, comment on pull request that they need to update to
  include most recent changes (and merge).
- Continue around room.
- Once everyone is in, have everyone pull new changes.
- Have everyone add email address.
- Repeat process of accepting pull requests, commenting, etc. from the other
  end of the room.

## Test Driven Development

### Interest Rate Calculator

The standard formula for continuous compounding interest is
`A = P * e ^ (r * t)` where `A` is the amount owed at time `t`, `e` is the
constant, `r` is the interest rate, and `P` is the principal. In the solution
below, we're just calculating the interest, not the amount owed.

{% highlight javascript %}
var calculateInterest = function(principal, rate, years) {
  if (rate < 0) { throw new Error('Cannot provide negative interest rates.'); }
  return principal * Math.exp(rate * years) - principal;
};

describe('calculateInterest()', function() {
  it('handles negative principal', function() {
    expect(calculateInterest(-10, 0.04, 1)).to.be.closeTo(-0.41, 0.01);
  });

  it('does not handle negative interest rates', function() {
    expect(function() { calculateInterest(1, -0.2, 1) }).to.throw(/negative interest/);
  });

  it('calculates interest for $1000 at 4% over 3 years', function() {
    expect(calculateInterest(1000, 0.04, 3)).to.be.closeTo(127.50, 0.01);
  });
});
{% endhighlight %}


### Gravity Challenge

Here's one solution to the gravity challenge. Their solutions should run
through Travis, but that could take a while.

{% highlight javascript %}
var fallingTime = function(distance) {
  if (distance < 0) { throw new Error('cannot fall negative distances'); }
  return Math.sqrt(distance * 2 / 9.81);
};

describe('fallingTime()', function() {
  it('takes no time to fall when already on the ground', function() {
    expect(fallingTime(0)).to.eql(0);
  });

  it('calculates time to fall 10 meters', function() {
    expect(fallingTime(10)).to.be.closeTo(1.4278, 0.0001);
  });

  it('raises for negative distances', function() {
    expect(function() { fallingTime(-4) }).to.throw(/negative distance/);
  });
});
{% endhighlight %}


## Abstractions

Go back to the fruits code from the previous day. Basically, we just need
to refactor to take another argument, `fn` that replaces the calls to
`indicateFruitThatNeedsPurchasing` and/or `indicateFruitPurchased`.

{% highlight javascript %}
var continueIterating = function(array, n, fn) {
  if (n < array.length) {
    fn(array[n]);
    continueIterating(array, n+1, fn);
  }
};

var startIterating = function(array, fn) {
  continueIterating(array, 0, fn);
};

startIterating(fruits, indicateFruitThatNeedsPurchasing);
startIterating(fruits, indicateFruitPurchased);
{% endhighlight %}


After that, all that needs to be done is some renaming and moving functions
around. The function `startIterating` becomes `each` and the function
`continueIterating` becomes `next` and gets moved into the `each` function.

{% highlight javascript %}
var each = function(array, fn) {
  var next = function(array, n, fn) {
    if (n < array.length) {
      fn(array[n]);
      next(array, n+1, fn);
    }
  };
  next(array, 0, fn);
};

each(fruits, indicateFruitThatNeedsPurchasing);
each(fruits, indicateFruitPurchased);
{% endhighlight %}