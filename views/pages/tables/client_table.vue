<template>
  <div>
    <v-row no-gutters>
      <v-col cols="12">
        <v-col v-if="loading" cols="12" class="d-flex justify-center">
          <!-- Placeholder for loading state -->
          <v-progress-circular
            indeterminate
            color="primary"
            size="70"
            width="7"
            class="mx-auto"
          ></v-progress-circular>
        </v-col>

        <!-- Error Message -->
        <v-col v-if="error" cols="12">
          <v-alert type="error" border="left" dismissible>
            {{ error }}
          </v-alert>
        </v-col>

        <!-- Result Message -->
        <v-col v-if="result_d.msg" cols="12">
          <v-alert closable type="success">
            {{ result_d.msg }}
          </v-alert>
        </v-col>
      </v-col>
    </v-row>

    <v-row no-gutters>
      <v-col cols="12">
        <!-- Data Table -->
        <v-col v-if="!loading && !error && clients.length">
          <v-card>
            <v-card-text class="w-50 float-right d-flex ga-3">
              <v-text-field
                v-model="search"
                label="Search"
                variant="outlined"
                hide-details
                single-line
              ></v-text-field>
              <v-btn size="large" class="mt-1" @click="showDetails()">+</v-btn>
            </v-card-text>

            <v-data-table
              :headers="headers"
              :items="filteredClient"
              item-key="clientCode"
            >
              <template v-slot:item="props">
                <tr>
                  <td>{{ props.item.clientCode }}</td>
                  <td>{{ props.item.clientEmailId }}</td>
                  <td>{{ props.item.clientMobileNo }}</td>
                  <td>{{ props.item.branchCode }}</td>
                  <td>{{ props.item.branchEmail }}</td>
                  <td>{{ props.item.branchMobileNo }}</td>
                  <td>
                    <v-dialog max-width="800">
      <template v-slot:activator="{ props: activatorProps }">
        <v-btn
          v-bind="activatorProps"
          
        ><v-icon icon="ri-close-circle-fill"> </v-icon></v-btn>
      </template>
      <template v-slot:default="{ isActive }">
        <v-card title="Delete Client Detail">
          <template v-slot:text>

           <p>Are You Sure Delete this Client Details?</p>

          </template>

          <v-card-actions>
            <v-spacer></v-spacer>

            <v-btn
              text="No Delete, it"
              variant="text"
              @click="isActive.value = false"
            ></v-btn>

            <v-btn
              color="surface-variant"
              text="Yes Delete, it"
              variant="flat"
              @click="deleteClient(props.item.clientCode)"
            ></v-btn>
          </v-card-actions>
        </v-card>
      </template>
    </v-dialog>
                  </td>
                </tr>
              </template>
            </v-data-table>
          </v-card>
        </v-col>
      </v-col>
    </v-row>

    <div v-if="sheet" class="overlay" @click="closeDetails"></div>
    <div :class="['right-sheet', { 'right-sheet--active': sheet }]" class="pa-2 bg-surface w-25">
      <h1 class="text-center">ADD CLIENT</h1>
      <form @submit.prevent="handleSubmit" class="mt-10">
        <v-text-field
        class="mb-5"
          v-model="ClientCode
"
          label="CLIENT CODE"
          :error-messages="validationErrors.ClientCode
"
          :rules="[v => !!v || 'Client Code is required']"
        ></v-text-field>
        <v-text-field
        class="mb-5"
          v-model="BranchCode
"
          label="BRANCH CODE"
          :error-messages="validationErrors.BranchCode
"
          :rules="[v => !!v || 'Branch Code is required']"
        ></v-text-field>
        <v-text-field
        class="mb-5"
          v-model="ClientEmailId
"
          label="CLIENT EMAIL"
          :error-messages="validationErrors.ClientEmailId
"
          :rules="[v => !!v || 'Client Email is required', v => /.+@.+\..+/.test(v) || 'E-mail must be valid']"
        ></v-text-field>
        <v-text-field
        class="mb-5"
          v-model="BranchEmail
"
          label="BRANCH E-MAIL"
          :error-messages="validationErrors.BranchEmail
"
          :rules="[v => !!v || 'Branch Email is required', v => /.+@.+\..+/.test(v) || 'E-mail must be valid']"
        ></v-text-field>
        <v-text-field
          class="mb-5"
          v-model="BranchMobile"
          label="BRANCH MOBILE"
          :error-messages="validationErrors.BranchMobile"
          :rules="[
            v => !!v || 'Branch mobile is required',
            v => /^[0-9]*$/.test(v) || 'Branch mobile must be numeric'
          ]"
        ></v-text-field>
        <v-text-field
          class="mb-5"
          v-model="ClientMobileno"
          label="CLIENT MOBILE"
          :error-messages="validationErrors.ClientMobileno"
          :rules="[
            v => !!v || 'Client mobile is required',
            v => /^[0-9]*$/.test(v) || 'Branch mobile must be numeric'
          ]"
        ></v-text-field>
       <div class="w-100 d-flex justify-center ga-3">
        <v-btn type="submit" :disabled="isSubmitting">Add</v-btn>
        <v-btn class="bg-info"  @click="closeDetails()">Close</v-btn>
       </div>
      </form>
    </div>
  </div>
</template>

<script setup>
import { computed, ref } from 'vue';

// Define reactive state variables
const search = ref('');
const loading = ref(true);
const error = ref(null);
const clients = ref([]);

// Headers for the data table
const headers = [
  { title: 'Client Code', value: 'clientCode', sortable: true },
  { title: 'Client email', value: 'clientEmailID', sortable: true },
  { title: 'Client Mobileno', value: 'ClientMobileno', sortable: true },
  { title: 'Branch Code', value: 'branchCode', sortable: true },
  { title: 'Branch Email', value: 'branchEmail', sortable: true },
  { title: 'Branch Mobile', value: 'branchMobile', sortable: true },

 
  { title: 'Action', value: 'action', sortable: false },
];

// API URLs and keys
const apiUrl = 'https://g1.gwcindia.in/powerstocks/powerstocks-client-data.php';
const authKey = '0a7cb27e52927eacabbb7ecc738b0fea50b3967945257c43a67eb753cb465bd0';

// Fetch data from API
const fetchData = async () => {
  const formData = new FormData();
  formData.append('authKey', authKey);

  try {
    const response = await fetch(apiUrl, {
      method: 'POST',
      body: formData
    });

    if (!response.ok) {
      throw new Error('Network response was not ok');
    }

    clients.value = await response.json();
  } catch (err) {
    error.value = err.message;
  } finally {
    loading.value = false;
  }
};

// Filtered clients based on search term
const filteredClient = computed(() => {
  if (!search.value) return clients.value;

  return clients.value.filter(client =>
    client.clientCode.toLowerCase().includes(search.value.toLowerCase()) ||
    client.branchCode.toLowerCase().includes(search.value.toLowerCase()) ||
    client.branchEmail.toLowerCase().includes(search.value.toLowerCase())
  );
});

// Fetch data when the component is mounted
fetchData();

// Placeholder function for showing details
const showDetails = () => {
  // Implement your logic here
  sheet.value = true;
};
const closeDetails = () => {
  sheet.value = false;
  fetchData();
};
// Placeholder function for deleting a client



//add client data
const result_d = ref({});
const error_d = ref(null);
const loading_d = ref(false);
const ClientCode= ref('');
const BranchCode= ref('');
const BranchEmail= ref('');
const ClientEmailId= ref('');
const BranchMobile = ref('');
const ClientMobileno = ref('');
const isSubmitting = ref(false);
const sheet = ref(false);
// Validation errors
const validationErrors = ref({
  ClientCode: [],
  BranchCode: [],
  BranchEmail: [],
  ClientEmailId:[],
  BranchMobile:[],
  ClientMobileno:[]

});

// Form submission handler
const handleSubmit = async () => {
  isSubmitting.value = true;
  validationErrors.value = {
    ClientCode: [],
    BranchCode: [],
    BranchEmail: [],
    ClientEmailId:[],
    BranchMobile:[],
    ClientMobileno:[]
  };

  // Validate fields
  if (!ClientCode.value) validationErrors.value.ClientCode.push('Client Code is required');
  if (!BranchCode.value) validationErrors.value.BranchCode.push('Branch Code is required');
  if (!BranchEmail.value) validationErrors.value.BranchEmail.push('Branch Email is required');
  if (BranchEmail.value && !/.+@.+\..+/.test(BranchEmail.value)) validationErrors.value.BranchEmail.push('E-mail must be valid');
  if (!ClientEmailId.value) validationErrors.value.ClientEmailId.push('Branch Email is required');
  if (ClientEmailId.value && !/.+@.+\..+/.test(ClientEmailId.value)) validationErrors.value.ClientEmailId.push('E-mail must be valid');
  
  if (!BranchMobile.value) validationErrors.value.BranchMobile.push('Branch Mobile no is required');
  if (BranchMobile.value && !/^[0-9]*$/.test(BranchMobile.value)) validationErrors.value.BranchMobile.push('Number must be valid');

  if (!ClientMobileno.value) validationErrors.value.ClientMobileno.push('Client Mobile is required');
  if (ClientMobileno.value && !/^[0-9]*$/.test(ClientMobileno.value)) validationErrors.value.ClientMobileno.push('Number must be valid');

  // If there are validation errors, stop form submission
  if (validationErrors.value.ClientCode.length || validationErrors.value.BranchCode.length || validationErrors.value.BranchEmail.length || validationErrors.value.ClientEmailId.length || validationErrors.value.BranchMobile.length|| validationErrors.value.ClientMobileno.length) {
     isSubmitting.value = false;
     return;
  }

  const formData = new FormData();
  formData.append('ClientCode',ClientCode.value);
  formData.append('BranchCode', BranchCode.value);
  formData.append('BranchEmail', BranchEmail.value);
  formData.append('ClientEmailId', ClientEmailId.value);
  formData.append('BranchMobile', BranchMobile.value);
  formData.append('ClientMobileno', ClientMobileno.value);

  try {
    const response_d = await fetch('https://g1.gwcindia.in/powerstocks/powerstocks-client-entry.php', {
      method: 'POST',
      body: formData,
    
    });

    if (!response_d.ok) {
      throw new Error('Network response was not ok');
    }

    result_d.value = await response_d.json();
    // Close the dialog on successful submission
  } catch (err) {
    error_d.value = err.message;
  } finally {
    isSubmitting.value = false;
    fetchData();
  }
};


//delete data.....
const error_r = ref(null);
const loading_r = ref(false);
const deleteurl= 'https://g1.gwcindia.in/powerstocks/powerstocks-remove-client.php'
const deleteClient= async(clientCode)=>{
 
  const formData = new FormData();
  formData.append('authKey', authKey);
  formData.append('clientCode', clientCode);

  try {
    const response_r = await fetch(deleteurl, {
      method: 'POST',
      body: formData
    });

    if (!response_r.ok) {
      throw new Error('Network response was not ok');
    }

    clients.value = await response_r.json();
  } catch (err) {
    error_r.value = err.message;
  } finally {
    loading_r.value = false;
    fetchData();
  }

}
</script>

<style scoped>
/* Add your styles here */
.right-sheet {
  position: fixed;
  z-index: 1001; /* Ensure this is above the overlay */
  block-size: 100%;
  inset-block-start: 0;
  inset-inline-end: -100%;
  transition: inset-inline-end 0.3s ease;
}

.right-sheet--active {
  inset-inline-end: 0;
}

.overlay {
  position: fixed;
  z-index: 1000; /* Ensure this is below the right-sheet */
  background-color: rgba(0, 0, 0, 50%);
  block-size: 100%;
  inline-size: 100%;
  inset-block-start: 0;
  inset-inline-start: 0;
  transition: opacity 0.3s ease;
}

#loading, #error, #result {
  padding: 10px;
  border: 1px solid #ccc;
  border-radius: 4px;
  background-color: #f9f9f9;
  margin-block-start: 20px;
}

#error {
  color: red;
}
</style>
