# vue-store
*still developing...*

a simpler way to use redux-like state management in vue project

* Simpler way to write your mutations, getters, actions, no ACTION_TYPE, commit, dispatch
* Better and natural support for typescript
* Only 1.5KB after gzip but with adequate support

### install
```bash
npm install @tingyuan/vue-store
```

### example
`store.ts`
```typescript
type Item = { text: string, done: boolean }

const todoModule = {
  list: [] as Item[],
  get doneCount() {
    return this.list.filter(v => v.done).length
  },
  setList(list: Item[]) {
    this.list = list
  },
  addItem(text: string) {
    this.list.push({
      text, done: false
    })
  },
  async $fetchList() {
    const list = await request('/api/get-list')
    this.setList(list)
  }
}

const modules = {
  user: {
    name: 'nobody'
  },
  setUserName(username: string) {
    this.user.name = username
  },
  Todo: todoModule
}

import Vue from 'vue'
import VueStore from '@tingyuan/vue-store'

Vue.use(VueStore)

const store = VueStore.createStore(modules, {
  strict: true,
  plugins: [
    store => {
      store.subscribe(({type, actionType, payload}, state) => {
        if (type) {
          console.log('mutation called: ' + type)
        } else {
          console.log('action called: ' + actionType)
        }
      })
    }
  ]
})

store.Todo.doneCount // equals to `store.getters['Todo/doneCount']`
store.Todo.addItem('new item') // equals to `store.commit('Todo/addItem', 'new item')`
store.Todo.$fetchList() // equals to `store.dispatch('Todo/$fetchList')`

export default store
```

### api
* `store = VueStore.createStore(modules, options)`
* `store.watch(getter, callback)`
* `store.subscribe(listener)`
* `store.addModule(modulePath, module, options)`
* `store.removeModule(modulePath)`
* `store.replaceState(newState)`
* `store.getState()`
* `store.hotUpdate(path, module)`

### convention(compulsory in fact)
1. name of sub-module(namespace) starts with **capital letter**
2. name of action method(with side-effect) starts with **'$'**
3. getter property will be taken as 'getters'
