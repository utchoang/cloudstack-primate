# Testing with Jest
Test single component using  vue-test-utils and jest to run unit tests

## Run Specs Locally
``` bash
# run all specs
$ npm run test:unit
```

## Configuration
Jest can be configured through the jest.configUtil.js file in the root directory of project.

## Mockups
- `mockStore.js` Using mock options to overwrite `$store`.
- `mockI18n.js` Using mock options to overwrite `$t`.
- `mockRouter.js` Using mock options to overwrite `$route` and `$router`.
- `mockAxios.js` Using mock options to overwrite `axios` request.

## Test single-component
Refer: https://vuejs.org/v2/guide/unit-testing.html

- Creates a Wrapper that contains the mounted and rendered Vue component. Example:
```javascript
import { mount } from '@vue/test-utils'
import AutogenView from '@/views/AutogenView'

const wrapper = mount(AutogenView)
```

- To mock `props` data:
```javascript
const wrapper = mount(ActionButton, {
  propsData: {
    dataView: true,
    resource: {
      id: 'test-resource-id',
      state: 'Stopped'
    }
  }
})
```

- To mock `data`:
```javascript
const wrapper = mount(ActionButton, {
  data () {
    return {
      key: 'test-value'
    }
  }
})
```

- To mock `method`:
```javascript
const testMethod = jest.fn()
const wrapper = mount(AutogenView)
wrapper.setMethods({ testMethod })
```

- To mock `$store`:
```javascript
import { localVue, mockStore } from 'setup'

const state = {
  user: {
    apis: {
      'testApiName': {},
    }
  }
}
const store = mockStore.mock(state)

const wrapper = mount(ActionButton, {
  localVue,
  store
})
```

- To mock `i18n`:
```javascript
import { localVue, mockI18n } from 'setup'

const messages = {
  en: { name: 'test-name-en' },
  de: { name: 'test-name-de' }
}
const i18n = mockI18n.mock('en', messages)

const wrapper = mount(ActionButton, {
  localVue,
  i18n
})
```

- To mock `$route` and `$router`:
```javascript
import { localVue, mockRouter } from 'setup'

const routes = [
  {
    name: 'testRouter1',
    path: '/test-router-1',
    meta: {}
  },
  {
    name: 'testRouter2',
    path: '/test-router-2',
    meta: {}
  }
]
const router = mockRouter.mock(routes)

const wrapper = mount(ActionButton, {
  localVue,
  router
})

// To navigate to a different URL
router.push({ name: 'testRouter2' })
```

- To mock `axios`:
```javascript
import { localVue, mockAxios } from 'setup'
jest.mock('axios', () => mockAxios)

const dataMock = {
  testapiresponse: {
    count: 1,
    testapi: [{
      id: 'test-id'
    }]
  }
}

mockAxios.mockResolvedValue(dataMock)
// or mockAxios.mockRejectedValue(errors) for throw error from api

const wrapper = mount(ActionButton, {
  localVue,
  router
})
```

## Check the test results
Check the output or received of the component matches what is expected.

- Check the value of the props or data with the props or data expected
```javascript
const wrapper = mount(Component, {
  data () {
    return {
      key: 'test-value'
    }
  }
})

wrapper.vm.$nextTick(() => {
  expect(wrapper.vm.key).toEqual('test-value')
})
```

- Check if the method function is called or called with a specific parameter
```javascript
const testMethod = jest.fn()
const wrapper = mount(Component)
wrapper.setMethods({ testMethod })

wrapper.vm.$nextTick(() => {
  wrapper.vm.testMethod('test-value')

  expect(testMethod).toHaveBeenCalled()
  expect(testMethod).toHaveBeenCalledWith('test-value')
})
```

- Check if Axios API is called or called with a URL, parameter, method, etc.
```javascript
const dataMock = {}
mockAxios.mockResolvedValue(dataMock)
const wrapper = mount(Component)

wrapper.vm.$nextTick(() => {
  expect(mockAxios).toHaveBeenCalled()
  expect(mockAxios).toHaveBeenCalledWith({
    url: '/',
    method: 'GET',
    body: new URLSearchParams(),
    params: {
      command: 'testAPiName',
      response: 'json'
    }
  })
})
```

- Check the feedback data from the API. We need to use setTimeout() to wait for a response from the API and done() for callback response successfully. Example:
```javascript
it('Example chẹc api response', async (done) => {
  const dataMock = {
    testapinameresponse: {
      count: 1,
      testapi: [{
        id: 'test-id'
      }]
    }
  }
  mockAxios.mockResolvedValue(dataMock)
  const wrapper = mount(Component, {
    data () { 
      return { 
        testData: [] 
      }
    }
  })
  await wrapper.vm.$nextTick()
  await wrapper.vm.fetchData()

  setTimeout(() => {
    expect(wrapper.vm.testData).toEqual([{
      id: 'test-id'
    }])

    done()
  })
})
```