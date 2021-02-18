# Redux Saga Test Plan

## Notice:
* v4.0.0-rc.1 compatible with redux-saga prior v1.1.1
* v4.0.0-rc.2 compatible with redux-saga v1.1.1 and typescript 3.6 onwards.

[![npm](https://img.shields.io/npm/v/redux-saga-test-plan.svg?style=flat-square)](https://www.npmjs.com/package/redux-saga-test-plan)
[![Travis branch](https://img.shields.io/travis/jfairbank/redux-saga-test-plan/master.svg?style=flat-square)](https://travis-ci.org/jfairbank/redux-saga-test-plan)
[![Codecov](https://img.shields.io/codecov/c/github/jfairbank/redux-saga-test-plan.svg?style=flat-square)](https://codecov.io/gh/jfairbank/redux-saga-test-plan)

#### Test Redux Saga with an easy plan.

Redux Saga Test Plan makes testing sagas a breeze. Whether you need to test
exact effects and their ordering or just test your saga `put`'s a specific
action at some point, Redux Saga Test Plan has you covered.


Redux Saga Test Plan 使测试 Sagas 变得轻而易举。你可以选择测试指定的 effects 及他们的顺序逻辑，或者只是测试某处的 `put` action，Redux Saga Test Plan 完全满足你

Redux Saga Test Plan aims to embrace both integration testing and unit testing
approaches to make testing your sagas easy.


Redux Saga Test Plan 旨在涵盖集成测试和单元测试，使 Sagas测试变得更容易。

## Table of Contents

- [Redux Saga Test Plan](#redux-saga-test-plan)
  - [Notice:](#notice)
      - [Test Redux Saga with an easy plan.](#test-redux-saga-with-an-easy-plan)
  - [Table of Contents](#table-of-contents)
  - [Documentation](#documentation)
  - [Integration Testing / 集成测试](#integration-testing--集成测试)
    - [Simple Example / 简单的例子](#simple-example--简单的例子)
    - [Mocking with Providers / 模拟 Providers](#mocking-with-providers--模拟-providers)
    - [Example with Reducer](#example-with-reducer)
  - [Unit Testing / 单元测试](#unit-testing--单元测试)
  - [Install](#install)
    - [Redux Saga 1.0 Beta](#redux-saga-10-beta)
    - [Redux Saga Stable](#redux-saga-stable)

## Documentation

* [Introduction](http://redux-saga-test-plan.jeremyfairbank.com/)
* [Getting Started](http://redux-saga-test-plan.jeremyfairbank.com/getting-started.html)
* [Integration Testing](http://redux-saga-test-plan.jeremyfairbank.com/integration-testing/)
* [Unit Testing](http://redux-saga-test-plan.jeremyfairbank.com/unit-testing/)

## Integration Testing / 集成测试

**Requires global `Promise` to be available**

**需要全局有 `Promise`**

One downside to unit testing sagas is that it couples your test to your
implementation. Simple reordering of yielded effects in your saga could break
your tests even if the functionality stays the same. If you're not concerned
with the order or exact effects your saga yields, then you can take an
integrative approach, testing the behavior of your saga when run by Redux Saga.
Then, you can simply test that a particular effect was yielded during the saga
run. For this, use the `expectSaga` test function.

Sagas 单元测试的缺点之一是它将测试与实现的代码联系起来了。Saga 中 yield 的 effect 稍微调整下顺序可能就会使测试失败，尽管要测的功能保持不变，即测试用例本身的逻辑不需要改变。如果你不关心 yield effect 的顺序或指定的 effect yield 的返回值，那么你可以使用集成测试，测试这个行为由 Redux Saga 本身来运行。然后，您可以简单地测试在 Saga 中是否 yield 了某个 effect。要做到这一点，可以使用 `expectSaga` 测试函数。

### Simple Example / 简单的例子

Import the `expectSaga` function and pass in your saga function as an argument.
Any additional arguments to `expectSaga` will become arguments to the saga
function. The return value is a chainable API with assertions for the different
effect creators available in Redux Saga.


导入 `expectSaga` 函数并将 saga 函数作为参数传递。`expectSaga` 的任何其他参数将成为 saga 函数的参数。返回值是一个可链式调用的API，其中包含 Redux Saga 中不同的可用的 effect creators。

In the example below, we test that the `userSaga` successfully `put`s a
`RECEIVE_USER` action with the `fakeUser` as the payload. We call `expectSaga`
with the `userSaga` and supply an `api` object as an argument to `userSaga`. We
assert the expected `put` effect via the `put` assertion method. Then, we call
the `dispatch` method with a `REQUEST_USER` action that contains the user id
payload. The `dispatch` method will supply actions to `take` effects. Finally,
we start the test by calling the `run` method which returns a `Promise`. Tests
with `expectSaga` will always run asynchronously, so the returned `Promise`
resolves when the saga finishes or when `expectSaga` forces a timeout. If you're
using a test runner like Jest, you can return the `Promise` inside your Jest
test so Jest knows when the test is complete.

在下面的示例中，我们测试了 `userSaga` 是否成功 `put` 了一个 `payload` 为 `fakeUser`、`type` 为 `RECEIVE_USER` 的 `action`。调用  `expectSaga` 函数，`userSaga` 函数作为第一个参数，并提供一个 `api` 对象作为第二个参数，它将会是 `userSaga` 的参数。我们
通过 `put` 断言方法来断言预期的 `put` effect 的值。然后，我们调用 dispatch 断言方法来断言 saga 中是否 dispatch 过 type 为 REQUEST_USER、payload 为 user id 的 action。 `dispatch` 方法将为 `take` effect 提供 action。最后，我们通过调用将返回 Promise 的run方法开始测试。`expectSaga` 函数是异步运行的，因此返回的 `Promise` 解决传奇故事何时结束或`expectSaga`强制超时的问题。如果你是
使用像Jest这样的测试运行器，您可以在Jest中返回`Promise`
测试，以便Jest知道测试何时完成。

```js
import { call, put, take } from 'redux-saga/effects';
import { expectSaga } from 'redux-saga-test-plan';

function* userSaga(api) {
  const action = yield take('REQUEST_USER');
  const user = yield call(api.fetchUser, action.payload);

  yield put({ type: 'RECEIVE_USER', payload: user });
}

it('just works!', () => {
  const api = {
    fetchUser: id => ({ id, name: 'Tucker' }),
  };

  return expectSaga(userSaga, api)
    // Assert that the `put` will eventually happen.
    .put({
      type: 'RECEIVE_USER',
      payload: { id: 42, name: 'Tucker' },
    })

    // Dispatch any actions that the saga will `take`.
    .dispatch({ type: 'REQUEST_USER', payload: 42 })

    // Start the test. Returns a Promise.
    .run();
});
```

### Mocking with Providers / 模拟 Providers

`expectSaga` runs your saga with Redux Saga, so it will try to resolve effects
just like Redux Saga would in your application. This is great for integration
testing, but sometimes it can be laborious to bootstrap your entire application
for tests or mock things like server APIs. In those cases, you can use
_providers_ which are perfect for mocking values directly with `expectSaga`.
Providers are similar to middleware that allow you to intercept effects before
they reach Redux Saga. You can choose to return a mock value instead of allowing
Redux Saga to handle the effect, or you can pass on the effect to other
providers or eventually Redux Saga.


`expectSaga` 借助 Redux Saga 来运行 Saga，因此它将尝试解决效果
就像Redux Saga在您的应用程序中一样。这非常适合集成
测试，但是有时引导您的整个应用程序可能很麻烦
用于测试或模拟服务器API之类的东西。在这种情况下，您可以使用
_providers_非常适合直接通过`expectSaga`模拟值。
提供程序类似于中间件，使您可以在之前拦截效果
他们到达Redux Saga。您可以选择返回模拟值，而不是允许
Redux Saga可以处理效果，也可以将效果传递给其他人
提供者或最终Redux Saga。

`expectSaga` has two flavors of providers, _static providers_ and _dynamic
providers_. Static providers are easier to compose and reuse, but dynamic
providers give you more flexibility with non-deterministic effects. Here is one
example below using static providers. There are more examples of providers [in
the
docs](http://redux-saga-test-plan.jeremyfairbank.com/integration-testing/mocking/).


`expectSaga`具有两种类型的提供程序：_static provider_和_dynamic
provider_。静态提供者更容易组成和重用，而动态提供者
提供程序为您提供更多不确定性效果，带来更大的灵活性。这是一个
下面的示例使用静态提供程序。提供者的更多示例[在
的
docs]（http://redux-saga-test-plan.jeremyfairbank.com/integration-testing/mocking/）

```js
import { call, put, take } from 'redux-saga/effects';
import { expectSaga } from 'redux-saga-test-plan';
import * as matchers from 'redux-saga-test-plan/matchers';
import { throwError } from 'redux-saga-test-plan/providers';
import api from 'my-api';

function* userSaga(api) {
  try {
    const action = yield take('REQUEST_USER');
    const user = yield call(api.fetchUser, action.payload);
    const pet = yield call(api.fetchPet, user.petId);

    yield put({
      type: 'RECEIVE_USER',
      payload: { user, pet },
    });
  } catch (e) {
    yield put({ type: 'FAIL_USER', error: e });
  }
}

it('fetches the user', () => {
  const fakeUser = { name: 'Jeremy', petId: 20 };
  const fakeDog = { name: 'Tucker' };

  return expectSaga(userSaga, api)
    .provide([
      [call(api.fetchUser, 42), fakeUser],
      [matchers.call.fn(api.fetchPet), fakeDog],
    ])
    .put({
      type: 'RECEIVE_USER',
      payload: { user: fakeUser, pet: fakeDog },
    })
    .dispatch({ type: 'REQUEST_USER', payload: 42 })
    .run();
});

it('handles errors', () => {
  const error = new Error('error');

  return expectSaga(userSaga, api)
    .provide([
      [matchers.call.fn(api.fetchUser), throwError(error)],
    ])
    .put({ type: 'FAIL_USER', error })
    .dispatch({ type: 'REQUEST_USER', payload: 42 })
    .run();
});
```

Notice we pass in an array of tuple pairs (or array pairs) that contain a
matcher and a fake value. You can use the effect creators from Redux Saga or
matchers from the `redux-saga-test-plan/matchers` module to match effects. The
bonus of using Redux Saga Test Plan's matchers is that they offer special
partial matchers like `call.fn` which matches by the function without worrying
about the specific `args` contained in the actual `call` effect. Notice in the
second test that we can also simulate errors with the `throwError` function from
the `redux-saga-test-plan/providers` module. This is perfect for simulating
server problems.


请注意，我们传入的元组对（或数组对）数组包含一个
匹配器和假值。您可以使用Redux Saga中的效果创建者或
来自“ redux-saga-test-plan / matchers”模块的匹配器以匹配效果。的
使用Redux Saga测试计划的匹配器的好处是它们提供了特殊的功能
像`call.fn`这样的部分匹配器，可以通过函数进行匹配而不必担心
关于实际`call`效果中包含的特定`args`。注意在
第二次测试，我们还可以使用以下命令中的`throwError`函数来模拟错误
`redux-saga-test-plan / providers`模块。这非常适合模拟
服务器问题。

### Example with Reducer

One good use case for integration testing is testing your reducer too. You can
hook up your reducer to your test by calling the `withReducer` method with your
reducer function.

集成测试的一个很好的用例是也测试您的reducer。您可以
通过在您的计算机上调用`withReducer`方法，将减速器连接到测试中
减速器功能。

```js
import { put } from 'redux-saga/effects';
import { expectSaga } from 'redux-saga-test-plan';

const initialDog = {
  name: 'Tucker',
  age: 11,
};

function reducer(state = initialDog, action) {
  if (action.type === 'HAVE_BIRTHDAY') {
    return {
      ...state,
      age: state.age + 1,
    };
  }

  return state;
}

function* saga() {
  yield put({ type: 'HAVE_BIRTHDAY' });
}

it('handles reducers and store state', () => {
  return expectSaga(saga)
    .withReducer(reducer)

    .hasFinalState({
      name: 'Tucker',
      age: 12, // <-- age changes in store state
    })

    .run();
});
```

## Unit Testing / 单元测试

If you want to ensure that your saga yields specific types of effects in a
particular order, then you can use the `testSaga` function. Here's a simple
example:


如果您想确保自己的 saga yield 会产生特定类型的效果，并且和顺序相关，则可以使用 `testSaga` 函数。这是一个简单的
例子：

```js
import { testSaga } from 'redux-saga-test-plan';

function identity(value) {
  return value;
}

function* mainSaga(x, y) {
  const action = yield take('HELLO');

  yield put({ type: 'ADD', payload: x + y });
  yield call(identity, action);
}

const action = { type: 'TEST' };

it('works with unit tests', () => {
  testSaga(mainSaga, 40, 2)
    // advance saga with `next()`
    .next()

    // assert that the saga yields `take` with `'HELLO'` as type
    .take('HELLO')

    // pass back in a value to a saga after it yields
    .next(action)

    // assert that the saga yields `put` with the expected action
    .put({ type: 'ADD', payload: 42 })

    .next()

    // assert that the saga yields a `call` to `identity` with
    // the `action` argument
    .call(identity, action)

    .next()

    // assert that the saga is finished
    .isDone();
});
```

## Install

### Redux Saga 1.0 Beta

Redux Saga Test Plan **v4.0.0-beta.1** supports Redux Saga **v1.0.0-beta.1**.

Install with yarn or npm.

```
yarn add redux-saga-test-plan@beta --dev
```

```
npm install --save-dev redux-saga-test-plan@beta
```

### Redux Saga Stable

Redux Saga Test Plan **v3** supports Redux Saga **>= v0.15**.

Install with yarn or npm.

```
yarn add redux-saga-test-plan --dev
```

```
npm install --save-dev redux-saga-test-plan
```
