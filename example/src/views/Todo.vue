<template>
  <div class="todo">
    <h1>todo list</h1>
    <input type="text" placeholder="enter whatever" v-model.trim="event" @keyup.enter="onAdd">
    <button @click="onAdd">add ({{$store.Todo.doneCount}}/{{$store.Todo.allCount}})</button>
    <span>
      <a href="javascript:void(0)"
        :class="{active: $store.Todo.status === 'all'}"
        @click="onFilter('all')">all
      </a> |
      <a href="javascript:void(0)"
        :class="{active: $store.Todo.status === 'done'}"
        @click="onFilter('done')">done
      </a> |
      <a href="javascript:void(0)"
        :class="{active: $store.Todo.status === 'undone'}"
        @click="onFilter('undone')">undo
      </a>
    </span>
    <div style="padding: 50px;">
      <transition-group name="slide-fade" tag="ol" v-if="list.length" class="list">
        <li v-for="item in list" :key="item.id" class="item">
          <span @click="onSwitch(item)"
            v-if="editId !== item.id"
            :class="{done: item.done}">
            {{item.text}}
          </span>
          <input type="text" autofocus v-else @change="onEdit" :value="item.text">
          &nbsp;
          <i @click="onDelete(item)">✗</i>&nbsp;
          <i @click="editId = editId ? 0 : item.id" v-if="!item.done">✒️</i>
        </li>
      </transition-group>
      <div v-else>
        empty list, please add
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import Vue from 'vue'
import store from '../store'
import todoModule from '../modules/todoListModule'

let removeTodo = store.addModule('Todo', todoModule)
let todoStore = store.Todo

console.log(333, store.getState())
removeTodo()
console.log(444, store.getState())
store.addModule('Todo', todoModule)

console.log(555, store.getState())

type Item = typeof todoModule.list[0]

type Status = Parameters<typeof todoModule.setStatus>[0]

// if (module.hot) {
//   module.hot.accept('@/modules/todoListModule', () => {
//     store.hotUpdate('Todo', require('@/modules/todoListModule').default)
//     todoStore = store.Todo
//   })
// }

// setTimeout(() => {
//   store.removeModule('Counter')
// }, 5000)

store.watch(() => {
  const a = store.Todo.doneCount && store.Todo.status
  return a
}, function () {
  console.log('done count change')
})

export default Vue.extend({
  data () {
    return {
      event: '',
      editId: 0
    }
  },
  computed: {
    list () {
      // console.log('todo computed', this.$store.Todo.displayList)
      return store.Todo.displayList
    }
  },
  methods: {
    onAdd () {
      if (this.event) {
        todoStore.add(this.event)
        this.event = ''
      }
    },
    onDelete (item: Item) {
      if (!item.done) {
        if (window.confirm(`Are you sure to delete: ${item.text} ?`)) {
          todoStore.remove(item.id)
        }
      } else {
        todoStore.remove(item.id)
      }
    },
    onSwitch (item: Item) {
      todoStore.markDone(item.id)
    },
    onFilter (type: Status) {
      todoStore.setStatus(type)
    },
    onEdit (evt: Event) {
      const newText = (evt.target as HTMLInputElement).value.trim()
      if (newText) {
        todoStore.edit({
          id: this.editId,
          text: newText
        })
      }
      this.editId = 0
    }
  },
  created () {
    todoStore.$fetchList()
  }
})
</script>

<style scoped>
.list {
  width: 50%;
  margin: 0 auto;
  text-align: left;
  list-style-position: inside;
}
.item {
  padding: 10px;
}
.done {
  text-decoration: line-through;
  font-style: italic;
}
.active {
  color: yellowgreen;
}
i {
  display: inline-block;
  cursor: pointer;
  width: 1em;
  height: 1em;
  vertical-align: middle;
}
.slide-fade-enter-active {
  transition: all .2s ease;
}
.slide-fade-leave-active {
  transition: all .4s cubic-bezier(1.0, 0.5, 0.8, 1.0);
  position: absolute;
}
.slide-fade-enter, .slide-fade-leave-to
/* .slide-fade-leave-active for below version 2.1.8 */ {
  transform: translateX(10px);
  opacity: 0;
}
.slide-fade-move {
  transition: all .5s;
}
</style>
