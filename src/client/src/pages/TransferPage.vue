<template>
  <q-page class="row justify-evenly q-my-lg">
    <div class="col-12 col-md-8 col-lg-6">
      <q-card class="q-pa-md" flat bordered style="min-width: 30em;">
        <q-card-section>
          <section class="q-mb-xl">
            <div class="text-h6">Transfer Money</div>
            <span class="text-caption">Transfer money to another account without limits!</span>
          </section>
          <q-form @submit="onSubmit" @reset="onReset" class="q-gutter-md">
            <q-select filled v-model="transferData.from_account_id" label="From Account *" lazy-rules
              :options="accounts.map(account => ({ label: account.id, value: account.id }))" hint="Select an account"
              :rules="[val => val !== null || 'Please select an account']" />

            <q-input filled v-model="transferData.amount" label="Amount *" type="number" lazy-rules
              :rules="[val => val !== null && val !== '' || 'Please enter the amount']" />

            <q-input filled v-model="transferData.description" label="Description *" hint="Enter a description" lazy-rules
              :rules="[val => val && val.length > 0 || 'Please enter a description']" />

            <q-input filled v-model="transferData.to_account_id" label="To Account ID *"
              hint="Enter the recipient's account ID" lazy-rules
              :rules="[val => val && val.length > 0 || 'Please enter the recipient\'s account ID']" />

            <div>
              <q-btn label="Submit" type="submit" color="primary" />
              <q-btn label="Reset" type="reset" color="primary" flat class="q-ml-sm" />
            </div>
          </q-form>
        </q-card-section>
      </q-card>
    </div>
  </q-page>
</template>

<script>
import { ref, onMounted } from 'vue'
import { Notify } from 'quasar'
import { api } from 'src/boot/axios';

export default {
  setup() {
    const accounts = ref([])
    const transferData = ref({
      amount: '',
      description: '',
      from_account_id: '',
      to_account_id: ''
    })

    onMounted(async () => {
      try {
        const accountsResponse = await api.get('/api/v1/accounts/me')
        accounts.value = accountsResponse.data.data.reverse()
      } catch (error) {
        console.error(error)
      }
    })

    return {
      accounts,
      transferData
    }
  },

  methods: {
    async onSubmit() {
      try {
        this.transferData.from_account_id = this.transferData.from_account_id.value
        const response = await api.post('/api/v1/transaction/create', this.transferData)
        if (response.status === 201) {
          Notify.create({
            message: 'Transfer successful',
            color: 'positive',
            icon: 'check',
          });
          this.onReset()
          this.$router.push('/transactions')
        } else {
          throw new Error(response.data.message)
        }
      } catch (error) {
        Notify.create({
          message: 'Transfer failed with error: ' + error.response.data.message,
          color: 'negative',
          icon: 'warning',
        });
      }
    },
    onReset() {
      this.transferData.amount = ''
      this.transferData.description = ''
      this.transferData.to_account_id = ''
    }
  },

  computed: {
  }
}
</script>