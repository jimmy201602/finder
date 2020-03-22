<template>
  <q-list v-for="(menuItem, index) in menuList" :key="index">
    <q-item clickable :active="menuItem.label === 'Outbox'" v-ripple>
      <q-item-section avatar>
        <q-icon :name="menuItem.icon" />
      </q-item-section>
      <q-item-section>
        {{ menuItem.label }}
      </q-item-section>
    </q-item>

    <q-separator v-if="menuItem.separator" />

  </q-list>
</template>

<script>
export default {
  name: 'emailshortcuts',
  data () {
    return {
      menuList: [
        {
          icon: 'inbox',
          label: 'Inbox',
          separator: true
        },
        {
          icon: 'send',
          label: 'Outbox',
          separator: false
        },
        {
          icon: 'delete',
          label: 'Trash',
          separator: false
        },
        {
          icon: 'error',
          label: 'Spam',
          separator: true
        },
        {
          icon: 'settings',
          label: 'Settings',
          separator: false
        },
        {
          icon: 'feedback',
          label: 'Send Feedback',
          separator: false
        },
        {
          icon: 'help',
          iconColor: 'primary',
          label: 'Help',
          separator: false
        }
      ]
    }
  }
}
</script>

<style scoped>

</style>
