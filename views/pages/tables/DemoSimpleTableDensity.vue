<template>
  <div>
    <!-- Tab Headers -->
    <div class="tab-headers bg-taborder">
      <div @click="activeTab = 'one'" :class="{ active: activeTab === 'one' }">
        Tab One
      </div>
      <div @click="activeTab = 'two'" :class="{ active: activeTab === 'two' }">
        Tab Two
      </div>
      <div @click="activeTab = 'three'" :class="{ active: activeTab === 'three' }">
        Tab Three
      </div>
    </div>

    <!-- Tab Content Sections -->
    <div class="tab-sections pa-1">
      <div class="pa-1" v-show="activeTab === 'one'" tap="one">
        <div class="w-100 mt-1 pa-1">
    <v-col cols="12" class="d-flex ga-2">
      <v-col cols="4">
        <form @submit.prevent="fetchData" class="d-flex ga-2 rounded-sm ml-4">
          <input ref="dateInput" type="text" class="pa-3 rounded-sm border" style="outline: none;" name="datetimes" />
          <div>
            <v-btn type="submit" size="large" class="mt-1">Filter</v-btn>
          </div>
        </form>
      </v-col>
      <v-col cols="8" class="d-flex ga-2">
        <v-text-field
          v-model="search"
          label="Search"
          variant="outlined"
          hide-details
          single-line
        ></v-text-field>

        <v-menu v-model="menu" :close-on-content-click="false" offset-y>
                <template v-slot:activator="{ props }">
                  <v-btn class="mt-1" size="large" v-bind="props">Toggle Columns</v-btn>
                </template>
                <v-list>
                  <v-list-item @click="toggleColumn('mobileNo')">
                    <v-list-item-title>{{ showMobileno ? 'Hide' : 'Show' }} Mobile Number</v-list-item-title>
                  </v-list-item>
                  <v-list-item @click="toggleColumn('receivedDate')">
                    <v-list-item-title>{{ showReceivedDate ? 'Hide' : 'Show' }} Received Date</v-list-item-title>
                  </v-list-item>
                  <v-list-item @click="toggleColumn('receivedTime')">
                    <v-list-item-title>{{ showReceivedTime ? 'Hide' : 'Show' }} Received Time</v-list-item-title>
                  </v-list-item>
                  <v-list-item @click="toggleColumn('plan')">
                    <v-list-item-title>{{ showPlan ? 'Hide' : 'Show' }} Plan</v-list-item-title>
                  </v-list-item>
                  <v-list-item @click="toggleColumn('action')">
                    <v-list-item-title>{{ showAction ? 'Hide' : 'Show' }} Action</v-list-item-title>
                  </v-list-item>
                </v-list>
              </v-menu>
              <v-btn class="mt-1" size="large" @click="exportDataToCsv">Import CSV File</v-btn>

      </v-col>
    </v-col>
    <v-col cols="12">
      <span v-if="loading" class="d-flex justify-center align-center">
        <v-progress-circular :size="70" :width="7" color="purple" indeterminate></v-progress-circular>
      </span>
      <span v-if="error">
        <v-alert density="compact" type="error">{{ error }}</v-alert>
      </span>
      <v-col cols="12" v-if="!loading && !error">
        <v-card flat>
          <v-data-table :headers="computedHeaders" :items="filteredProducts" item-key="rowId">
            <template v-slot:item="{ item }">
              <tr>
                      <td>{{ item.rowId }}</td>
                      <td>{{ item.ClientId }}</td>
                      <td>{{ item.clientName }}</td>
                      <td v-if="showMobileno">{{ item.mobileNo }}</td>
                      <td v-if="showReceivedDate">{{ item.receivedDate }}</td>
                      <td v-if="showReceivedTime">{{ item.receivedTime }}</td>
                      <td v-if="showPlan">{{ item.plan }}</td>
                      <td v-if="showAction">
                        <v-btn class="bg-logcolor" @click="viewDetails(item)"><v-icon icon="ri-eye-2-fill"></v-icon></v-btn>
                      </td>
                    </tr>
            </template>
          </v-data-table>
        </v-card>
      </v-col>
    </v-col>

    
          <!-- date filter view data -->
  <div v-if="sheet" class="overlay" @click="closeDetails"></div>
  <div :class="['right-sheet', { 'right-sheet--active': sheet }]" class="pa-2 bg-oninfo">
    <v-col cols="12">
            <span v-if="loading" class="d-flex justify-center align-center">
              <v-progress-circular :size="70" :width="7" color="purple" indeterminate></v-progress-circular>
            </span>
            <span v-if="error">
              <v-alert density="compact" type="error">{{ error }}</v-alert>
            </span>
            <v-col cols="12" v-if="!loading && !error">
              <v-row no-gutters>
                <v-col class="pa-1 d-flex text-secondary" style="overflow: hidden;" cols="12" md="4">      
                    <strong >Client Name:</strong><p class="ml-2">{{ selectedClient?.clientName }}</p>      
                </v-col>
                <v-col class="pa-1 d-flex text-secondary" style="overflow: hidden;" cols="12" md="4">     
                    <strong>Pan Number:</strong><p class="ml-2">{{ selectedClient?.panNo }}</p>    
                </v-col>
                <v-col class="pa-1 d-flex text-secondary" style="overflow: hidden;" cols="12" md="4">           
                    <strong>Branch:</strong><p class="ml-2">AUOXXXXXXB</p>               
                </v-col>
                <v-col class="pa-1 d-flex text-secondary" style="overflow: hidden;" cols="12" md="4">                 
                    <strong>Mobile Number:</strong><p class="ml-2">{{ selectedClient?.mobileNo }}</p>                
                </v-col>
                <v-col class="pa-1 d-flex text-secondary" style="overflow: hidden;" cols="12" md="4">
                    <strong>Email ID:</strong><p class="ml-2">{{ selectedClient?.emailId }}</p>
                </v-col>
                <v-col class="pa-1 d-flex text-secondary" style="overflow: hidden;" cols="12" md="4">
                    <strong>Branch Email ID:</strong><p class="ml-2">{{ selectedClient?.branchEmail }}</p>                 
                </v-col>
              </v-row>
              <hr>
              <v-col cols="12">
              <div class="tabs d-flex justify-center">
                <button class="text-secondary" :class="{ active: tabview === 'stocklist' }" @click="tabview = 'stocklist'">1 Calls</button>
                <button class="text-secondary" :class="{ active: tabview === 'communication' }" @click="tabview = 'communication'">2 Communication</button>
                <button class="text-secondary" :class="{ active: tabview === 'confirmedstatus' }" @click="tabview = 'confirmedstatus'">3 Confirmation</button>
                <button class="text-secondary" :class="{ active: tabview === 'existingorder' }" @click="tabview = 'existingorder'">4 Existing Order/ Payments status</button>
                <button class="text-secondary" :class="{ active: tabview === 'orderplaced' }" @click="tabview = 'orderplaced'">5 Payment / Order status</button>
              </div>
              <div class="tab-contentmain pa-3">
                <div v-if="tabview === 'stocklist'">
                  <v-table>
                    <thead class="bg-indigo">
                      <tr>
                        <th class="text-left text-secondary">
                          STOCK SYMBOL
                        </th>
                        <th class="text-left text-secondary">
                          QUANTITY
                        </th>
                        <th class="text-left text-secondary">
                          BUY/SELL
                        </th>
                        <th class="text-left text-secondary">
                          PLAN
                        </th>
                      </tr>
                    </thead>
                    <tbody>
                      <tr
                        v-for="item in stockList"
                        :key="item.stockName"
                      >
                        <td class="text-secondary">{{ item.stockName }}</td>
                        <td class="text-secondary">{{ item.quantity }}</td>
                        <td class="text-secondary">{{ item.buySell }}</td>
                        <td class="text-secondary">{{ item.plan }}</td>
                      </tr>
                    </tbody>
                  </v-table>
                </div>
                <div v-if="tabview === 'communication'">
                  <div class="w-100">
                    <p class="font-weight-bold text-h6 text-secondary">Power Stock to Email:</p>
                  <div class="w-100 pa-1">
                    <p class='text-secondary'><span class="pa-1">1,</span>Sent an Email to Customer Mail ID</p>
                    <div class="d-flex ga-4 ml-5">
                      <p class='text-secondary'>{{ recommendation_notify.emailNotifyLog.noteDEmail }}</p>
                      <p class='text-secondary'>{{ recommendation_notify.emailNotifyLog.noteDEmailSentTime }}</p>
                    </div>
                  </div>
                    
                    <div class="w-100 pa-1 mt-3">
                      <p class='text-secondary'><span class="pa-1">2,</span>Delivered Email</p>
                    <div class="d-flex ga-4 ml-5">
                      <p class='text-secondary'>{{ recommendation_notify.emailNotifyLog.noteDEmail }}</p>
                      <p class='text-secondary'>{{ recommendation_notify.emailNotifyLog.noteDEmailDelTime }}</p>
                    </div>
                    </div>

                    <div class="w-100 mt-3">
                      <p class="font-weight-bold text-h6 text-secondary">Power Stock to Email:</p>
                      <div class="w-100 pa-1">
                    <p class='text-secondary'><span class="pa-1">1,</span>Sent an Whatsapp to Customer Mail ID</p>
                    <div class="d-flex ga-4 ml-5">
                      <p class='text-secondary'>{{ selectedClient?.mobileNo}}</p>
                      <p class='text-secondary'>{{ recommendation_notify.emailNotifyLog.noteDEmailSentTime }}</p>
                    </div>
                  </div>
                    
                    <div class="w-100 pa-1 mt-3">
                      <p class='text-secondary'><span class="pa-1">2,</span>Delivered Whatsapp to Customer Mobile no</p>
                    <div class="d-flex ga-4 ml-5">
                      <p class='text-secondary'>{{ selectedClient?.mobileNo }}</p>
                      <p class='text-secondary'>{{ recommendation_notify.emailNotifyLog.noteDEmailDelTime }}</p>
                    </div>
                    </div>

                    </div>
                    
                    </div>
                </div>
                <div v-if="tabview === 'confirmedstatus'">
                  <p class="mt-3 text-secondary">Confirmed by Customer Mail ID at {{ confirm_status.confirmStatusDateTime }}</p>
                  <p class="mt-3 text-secondary">Confirmed by Customer Whatsapp at {{ confirm_status.confirmStatusDateTime }}</p>
                </div>
                <div v-if="tabview === 'existingorder'">
                  <p class="mt-3 text-secondary">Existing Order at   2024-08-14 11:29:50</p>
                  <p class="mt-3 text-secondary">Reason:</p>

                </div>
                <div v-if="tabview === 'orderplaced'">
                  <p class="mt-3 text-secondary">Order Placed at   2024-08-14 11:29:50</p>
                 <p class="mt-3 text-secondary">Reason:</p>
                </div>
              </div>
            </v-col>
            </v-col>
           </v-col>
  </div>
  </div>
      </div>
      <div class="pa-1" v-show="activeTab === 'two'" tap="two">
        <div class="w-100 mt-1 pa-1">
    <v-col cols="12" class="d-flex ga-2">
      <v-col cols="4">
        <form @submit.prevent="fetchData" class="d-flex ga-2 rounded-sm ml-4">
          <input ref="dateInput" type="text" class="pa-3 rounded-sm border" style="outline: none;" name="datetimes" />
          <div>
            <v-btn type="submit" size="large" class="mt-1">Filter</v-btn>
          </div>
        </form>
      </v-col>
      <v-col cols="8" class="d-flex ga-2">
        <v-text-field
          v-model="search"
          label="Search"
          variant="outlined"
          hide-details
          single-line
        ></v-text-field>

        <v-menu v-model="menu" :close-on-content-click="false" offset-y>
                <template v-slot:activator="{ props }">
                  <v-btn class="mt-1" size="large" v-bind="props">Toggle Columns</v-btn>
                </template>
                <v-list>
                  <v-list-item @click="toggleColumn('mobileNo')">
                    <v-list-item-title>{{ showMobileno ? 'Hide' : 'Show' }} Mobile Number</v-list-item-title>
                  </v-list-item>
                  <v-list-item @click="toggleColumn('receivedDate')">
                    <v-list-item-title>{{ showReceivedDate ? 'Hide' : 'Show' }} Received Date</v-list-item-title>
                  </v-list-item>
                  <v-list-item @click="toggleColumn('receivedTime')">
                    <v-list-item-title>{{ showReceivedTime ? 'Hide' : 'Show' }} Received Time</v-list-item-title>
                  </v-list-item>
                  <v-list-item @click="toggleColumn('plan')">
                    <v-list-item-title>{{ showPlan ? 'Hide' : 'Show' }} Plan</v-list-item-title>
                  </v-list-item>
                  <v-list-item @click="toggleColumn('action')">
                    <v-list-item-title>{{ showAction ? 'Hide' : 'Show' }} Action</v-list-item-title>
                  </v-list-item>
                </v-list>
              </v-menu>
              <v-btn class="mt-1" size="large" @click="exportDataToCsv">Import CSV File</v-btn>

      </v-col>
    </v-col>
    <v-col cols="12">
      <span v-if="loading" class="d-flex justify-center align-center">
        <v-progress-circular :size="70" :width="7" color="purple" indeterminate></v-progress-circular>
      </span>
      <span v-if="error">
        <v-alert density="compact" type="error">{{ error }}</v-alert>
      </span>
      <v-col cols="12" v-if="!loading && !error">
        <v-card flat>
          <v-data-table :headers="computedHeaders" :items="filteredProducts" item-key="rowId">
            <template v-slot:item="{ item }">
              <tr>
                      <td>{{ item.rowId }}</td>
                      <td>{{ item.ClientId }}</td>
                      <td>{{ item.clientName }}</td>
                      <td v-if="showMobileno">{{ item.mobileNo }}</td>
                      <td v-if="showReceivedDate">{{ item.receivedDate }}</td>
                      <td v-if="showReceivedTime">{{ item.receivedTime }}</td>
                      <td v-if="showPlan">{{ item.plan }}</td>
                      <td v-if="showAction">
                        <v-btn class="bg-logcolor" @click="viewDetails(item)"><v-icon icon="ri-eye-2-fill"></v-icon></v-btn>
                      </td>
                    </tr>
            </template>
          </v-data-table>
        </v-card>
      </v-col>
    </v-col>

    
          <!-- date filter view data -->
  <div v-if="sheet" class="overlay" @click="closeDetails"></div>
  <div :class="['right-sheet', { 'right-sheet--active': sheet }]" class="pa-2 bg-oninfo">
    <v-col cols="12">
            <span v-if="loading" class="d-flex justify-center align-center">
              <v-progress-circular :size="70" :width="7" color="purple" indeterminate></v-progress-circular>
            </span>
            <span v-if="error">
              <v-alert density="compact" type="error">{{ error }}</v-alert>
            </span>
            <v-col cols="12" v-if="!loading && !error">
              <v-row no-gutters>
                <v-col class="pa-1 d-flex text-secondary" style="overflow: hidden;" cols="12" md="4">      
                    <strong >Client Name:</strong><p class="ml-2">{{ selectedClient?.clientName }}</p>      
                </v-col>
                <v-col class="pa-1 d-flex text-secondary" style="overflow: hidden;" cols="12" md="4">     
                    <strong>Pan Number:</strong><p class="ml-2">{{ selectedClient?.panNo }}</p>    
                </v-col>
                <v-col class="pa-1 d-flex text-secondary" style="overflow: hidden;" cols="12" md="4">           
                    <strong>Branch:</strong><p class="ml-2">AUOXXXXXXB</p>               
                </v-col>
                <v-col class="pa-1 d-flex text-secondary" style="overflow: hidden;" cols="12" md="4">                 
                    <strong>Mobile Number:</strong><p class="ml-2">{{ selectedClient?.mobileNo }}</p>                
                </v-col>
                <v-col class="pa-1 d-flex text-secondary" style="overflow: hidden;" cols="12" md="4">
                    <strong>Email ID:</strong><p class="ml-2">{{ selectedClient?.emailId }}</p>
                </v-col>
                <v-col class="pa-1 d-flex text-secondary" style="overflow: hidden;" cols="12" md="4">
                    <strong>Branch Email ID:</strong><p class="ml-2">{{ selectedClient?.branchEmail }}</p>                 
                </v-col>
              </v-row>
              <hr>
              <v-col cols="12">
              <div class="tabs d-flex justify-center">
                <button class="text-secondary" :class="{ active: tabview === 'stocklist' }" @click="tabview = 'stocklist'">1 Calls</button>
                <button class="text-secondary" :class="{ active: tabview === 'communication' }" @click="tabview = 'communication'">2 Communication</button>
                <button class="text-secondary" :class="{ active: tabview === 'confirmedstatus' }" @click="tabview = 'confirmedstatus'">3 Confirmation</button>
                <button class="text-secondary" :class="{ active: tabview === 'existingorder' }" @click="tabview = 'existingorder'">4 Existing Order/ Payments status</button>
                <button class="text-secondary" :class="{ active: tabview === 'orderplaced' }" @click="tabview = 'orderplaced'">5 Payment / Order status</button>
              </div>
              <div class="tab-contentmain pa-3">
                <div v-if="tabview === 'stocklist'">
                  <v-table>
                    <thead class="bg-indigo">
                      <tr>
                        <th class="text-left text-secondary">
                          STOCK SYMBOL
                        </th>
                        <th class="text-left text-secondary">
                          QUANTITY
                        </th>
                        <th class="text-left text-secondary">
                          BUY/SELL
                        </th>
                        <th class="text-left text-secondary">
                          PLAN
                        </th>
                      </tr>
                    </thead>
                    <tbody>
                      <tr
                        v-for="item in stockList"
                        :key="item.stockName"
                      >
                        <td class="text-secondary">{{ item.stockName }}</td>
                        <td class="text-secondary">{{ item.quantity }}</td>
                        <td class="text-secondary">{{ item.buySell }}</td>
                        <td class="text-secondary">{{ item.plan }}</td>
                      </tr>
                    </tbody>
                  </v-table>
                </div>
                <div v-if="tabview === 'communication'">
                  <div class="w-100">
                    <p class="font-weight-bold text-h6 text-secondary">Power Stock to Email:</p>
                  <div class="w-100 pa-1">
                    <p class='text-secondary'><span class="pa-1">1,</span>Sent an Email to Customer Mail ID</p>
                    <div class="d-flex ga-4 ml-5">
                      <p class='text-secondary'>{{ recommendation_notify.emailNotifyLog.noteDEmail }}</p>
                      <p class='text-secondary'>{{ recommendation_notify.emailNotifyLog.noteDEmailSentTime }}</p>
                    </div>
                  </div>
                    
                    <div class="w-100 pa-1 mt-3">
                      <p class='text-secondary'><span class="pa-1">2,</span>Delivered Email</p>
                    <div class="d-flex ga-4 ml-5">
                      <p class='text-secondary'>{{ recommendation_notify.emailNotifyLog.noteDEmail }}</p>
                      <p class='text-secondary'>{{ recommendation_notify.emailNotifyLog.noteDEmailDelTime }}</p>
                    </div>
                    </div>

                    <div class="w-100 mt-3">
                      <p class="font-weight-bold text-h6 text-secondary">Power Stock to Email:</p>
                      <div class="w-100 pa-1">
                    <p class='text-secondary'><span class="pa-1">1,</span>Sent an Whatsapp to Customer Mail ID</p>
                    <div class="d-flex ga-4 ml-5">
                      <p class='text-secondary'>{{ selectedClient?.mobileNo}}</p>
                      <p class='text-secondary'>{{ recommendation_notify.emailNotifyLog.noteDEmailSentTime }}</p>
                    </div>
                  </div>
                    
                    <div class="w-100 pa-1 mt-3">
                      <p class='text-secondary'><span class="pa-1">2,</span>Delivered Whatsapp to Customer Mobile no</p>
                    <div class="d-flex ga-4 ml-5">
                      <p class='text-secondary'>{{ selectedClient?.mobileNo }}</p>
                      <p class='text-secondary'>{{ recommendation_notify.emailNotifyLog.noteDEmailDelTime }}</p>
                    </div>
                    </div>

                    </div>
                    
                    </div>
                </div>
                <div v-if="tabview === 'confirmedstatus'">
                  <p class="mt-3 text-secondary">Confirmed by Customer Mail ID at {{ confirm_status.confirmStatusDateTime }}</p>
                  <p class="mt-3 text-secondary">Confirmed by Customer Whatsapp at {{ confirm_status.confirmStatusDateTime }}</p>
                </div>
                <div v-if="tabview === 'existingorder'">
                  <p class="mt-3 text-secondary">Existing Order at   2024-08-14 11:29:50</p>
                  <p class="mt-3 text-secondary">Reason:</p>

                </div>
                <div v-if="tabview === 'orderplaced'">
                  <p class="mt-3 text-secondary">Order Placed at   2024-08-14 11:29:50</p>
                 <p class="mt-3 text-secondary">Reason:</p>
                </div>
              </div>
            </v-col>
            </v-col>
           </v-col>
  </div>
  </div>
      </div>
      <div class="pa-1" v-show="activeTab === 'three'" tap="three">
        <div class="w-100 mt-1 pa-1">
    <v-col cols="12" class="d-flex ga-2">
      <v-col cols="4">
        <form @submit.prevent="fetchData" class="d-flex ga-2 rounded-sm ml-4">
          <input ref="dateInput" type="text" class="pa-3 rounded-sm border" style="outline: none;" name="datetimes" />
          <div>
            <v-btn type="submit" size="large" class="mt-1">Filter</v-btn>
          </div>
        </form>
      </v-col>
      <v-col cols="8" class="d-flex ga-2">
        <v-text-field
          v-model="search"
          label="Search"
          variant="outlined"
          hide-details
          single-line
        ></v-text-field>

        <v-menu v-model="menu" :close-on-content-click="false" offset-y>
                <template v-slot:activator="{ props }">
                  <v-btn class="mt-1" size="large" v-bind="props">Toggle Columns</v-btn>
                </template>
                <v-list>
                  <v-list-item @click="toggleColumn('mobileNo')">
                    <v-list-item-title>{{ showMobileno ? 'Hide' : 'Show' }} Mobile Number</v-list-item-title>
                  </v-list-item>
                  <v-list-item @click="toggleColumn('receivedDate')">
                    <v-list-item-title>{{ showReceivedDate ? 'Hide' : 'Show' }} Received Date</v-list-item-title>
                  </v-list-item>
                  <v-list-item @click="toggleColumn('receivedTime')">
                    <v-list-item-title>{{ showReceivedTime ? 'Hide' : 'Show' }} Received Time</v-list-item-title>
                  </v-list-item>
                  <v-list-item @click="toggleColumn('plan')">
                    <v-list-item-title>{{ showPlan ? 'Hide' : 'Show' }} Plan</v-list-item-title>
                  </v-list-item>
                  <v-list-item @click="toggleColumn('action')">
                    <v-list-item-title>{{ showAction ? 'Hide' : 'Show' }} Action</v-list-item-title>
                  </v-list-item>
                </v-list>
              </v-menu>
              <v-btn class="mt-1" size="large" @click="exportDataToCsv">Import CSV File</v-btn>

      </v-col>
    </v-col>
    <v-col cols="12">
      <span v-if="loading" class="d-flex justify-center align-center">
        <v-progress-circular :size="70" :width="7" color="purple" indeterminate></v-progress-circular>
      </span>
      <span v-if="error">
        <v-alert density="compact" type="error">{{ error }}</v-alert>
      </span>
      <v-col cols="12" v-if="!loading && !error">
        <v-card flat>
          <v-data-table :headers="computedHeaders" :items="filteredProducts" item-key="rowId">
            <template v-slot:item="{ item }">
              <tr>
                      <td>{{ item.rowId }}</td>
                      <td>{{ item.ClientId }}</td>
                      <td>{{ item.clientName }}</td>
                      <td v-if="showMobileno">{{ item.mobileNo }}</td>
                      <td v-if="showReceivedDate">{{ item.receivedDate }}</td>
                      <td v-if="showReceivedTime">{{ item.receivedTime }}</td>
                      <td v-if="showPlan">{{ item.plan }}</td>
                      <td v-if="showAction">
                        <v-btn class="bg-logcolor" @click="viewDetails(item)"><v-icon icon="ri-eye-2-fill"></v-icon></v-btn>
                      </td>
                    </tr>
            </template>
          </v-data-table>
        </v-card>
      </v-col>
    </v-col>

    
          <!-- date filter view data -->
  <div v-if="sheet" class="overlay" @click="closeDetails"></div>
  <div :class="['right-sheet', { 'right-sheet--active': sheet }]" class="pa-2 bg-oninfo">
    <v-col cols="12">
            <span v-if="loading" class="d-flex justify-center align-center">
              <v-progress-circular :size="70" :width="7" color="purple" indeterminate></v-progress-circular>
            </span>
            <span v-if="error">
              <v-alert density="compact" type="error">{{ error }}</v-alert>
            </span>
            <v-col cols="12" v-if="!loading && !error">
              <v-row no-gutters>
                <v-col class="pa-1 d-flex text-secondary" style="overflow: hidden;" cols="12" md="4">      
                    <strong >Client Name:</strong><p class="ml-2">{{ selectedClient?.clientName }}</p>      
                </v-col>
                <v-col class="pa-1 d-flex text-secondary" style="overflow: hidden;" cols="12" md="4">     
                    <strong>Pan Number:</strong><p class="ml-2">{{ selectedClient?.panNo }}</p>    
                </v-col>
                <v-col class="pa-1 d-flex text-secondary" style="overflow: hidden;" cols="12" md="4">           
                    <strong>Branch:</strong><p class="ml-2">AUOXXXXXXB</p>               
                </v-col>
                <v-col class="pa-1 d-flex text-secondary" style="overflow: hidden;" cols="12" md="4">                 
                    <strong>Mobile Number:</strong><p class="ml-2">{{ selectedClient?.mobileNo }}</p>                
                </v-col>
                <v-col class="pa-1 d-flex text-secondary" style="overflow: hidden;" cols="12" md="4">
                    <strong>Email ID:</strong><p class="ml-2">{{ selectedClient?.emailId }}</p>
                </v-col>
                <v-col class="pa-1 d-flex text-secondary" style="overflow: hidden;" cols="12" md="4">
                    <strong>Branch Email ID:</strong><p class="ml-2">{{ selectedClient?.branchEmail }}</p>                 
                </v-col>
              </v-row>
              <hr>
              <v-col cols="12">
              <div class="tabs d-flex justify-center">
                <button class="text-secondary" :class="{ active: tabview === 'stocklist' }" @click="tabview = 'stocklist'">1 Calls</button>
                <button class="text-secondary" :class="{ active: tabview === 'communication' }" @click="tabview = 'communication'">2 Communication</button>
                <button class="text-secondary" :class="{ active: tabview === 'confirmedstatus' }" @click="tabview = 'confirmedstatus'">3 Confirmation</button>
                <button class="text-secondary" :class="{ active: tabview === 'existingorder' }" @click="tabview = 'existingorder'">4 Existing Order/ Payments status</button>
                <button class="text-secondary" :class="{ active: tabview === 'orderplaced' }" @click="tabview = 'orderplaced'">5 Payment / Order status</button>
              </div>
              <div class="tab-contentmain pa-3">
                <div v-if="tabview === 'stocklist'">
                  <v-table>
                    <thead class="bg-indigo">
                      <tr>
                        <th class="text-left text-secondary">
                          STOCK SYMBOL
                        </th>
                        <th class="text-left text-secondary">
                          QUANTITY
                        </th>
                        <th class="text-left text-secondary">
                          BUY/SELL
                        </th>
                        <th class="text-left text-secondary">
                          PLAN
                        </th>
                      </tr>
                    </thead>
                    <tbody>
                      <tr
                        v-for="item in stockList"
                        :key="item.stockName"
                      >
                        <td class="text-secondary">{{ item.stockName }}</td>
                        <td class="text-secondary">{{ item.quantity }}</td>
                        <td class="text-secondary">{{ item.buySell }}</td>
                        <td class="text-secondary">{{ item.plan }}</td>
                      </tr>
                    </tbody>
                  </v-table>
                </div>
                <div v-if="tabview === 'communication'">
                  <div class="w-100">
                    <p class="font-weight-bold text-h6 text-secondary">Power Stock to Email:</p>
                  <div class="w-100 pa-1">
                    <p class='text-secondary'><span class="pa-1">1,</span>Sent an Email to Customer Mail ID</p>
                    <div class="d-flex ga-4 ml-5">
                      <p class='text-secondary'>{{ recommendation_notify.emailNotifyLog.noteDEmail }}</p>
                      <p class='text-secondary'>{{ recommendation_notify.emailNotifyLog.noteDEmailSentTime }}</p>
                    </div>
                  </div>
                    
                    <div class="w-100 pa-1 mt-3">
                      <p class='text-secondary'><span class="pa-1">2,</span>Delivered Email</p>
                    <div class="d-flex ga-4 ml-5">
                      <p class='text-secondary'>{{ recommendation_notify.emailNotifyLog.noteDEmail }}</p>
                      <p class='text-secondary'>{{ recommendation_notify.emailNotifyLog.noteDEmailDelTime }}</p>
                    </div>
                    </div>

                    <div class="w-100 mt-3">
                      <p class="font-weight-bold text-h6 text-secondary">Power Stock to Email:</p>
                      <div class="w-100 pa-1">
                    <p class='text-secondary'><span class="pa-1">1,</span>Sent an Whatsapp to Customer Mail ID</p>
                    <div class="d-flex ga-4 ml-5">
                      <p class='text-secondary'>{{ selectedClient?.mobileNo}}</p>
                      <p class='text-secondary'>{{ recommendation_notify.emailNotifyLog.noteDEmailSentTime }}</p>
                    </div>
                  </div>
                    
                    <div class="w-100 pa-1 mt-3">
                      <p class='text-secondary'><span class="pa-1">2,</span>Delivered Whatsapp to Customer Mobile no</p>
                    <div class="d-flex ga-4 ml-5">
                      <p class='text-secondary'>{{ selectedClient?.mobileNo }}</p>
                      <p class='text-secondary'>{{ recommendation_notify.emailNotifyLog.noteDEmailDelTime }}</p>
                    </div>
                    </div>

                    </div>
                    
                    </div>
                </div>
                <div v-if="tabview === 'confirmedstatus'">
                  <p class="mt-3 text-secondary">Confirmed by Customer Mail ID at {{ confirm_status.confirmStatusDateTime }}</p>
                  <p class="mt-3 text-secondary">Confirmed by Customer Whatsapp at {{ confirm_status.confirmStatusDateTime }}</p>
                </div>
                <div v-if="tabview === 'existingorder'">
                  <p class="mt-3 text-secondary">Existing Order at   2024-08-14 11:29:50</p>
                  <p class="mt-3 text-secondary">Reason:</p>

                </div>
                <div v-if="tabview === 'orderplaced'">
                  <p class="mt-3 text-secondary">Order Placed at   2024-08-14 11:29:50</p>
                 <p class="mt-3 text-secondary">Reason:</p>
                </div>
              </div>
            </v-col>
            </v-col>
           </v-col>
  </div>
  </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import 'daterangepicker';
import 'daterangepicker/daterangepicker.css';
import moment from 'moment';
import { computed, onMounted, ref } from 'vue';

// Define the active tab state
const activeTab = ref('one')


const dateInput = ref(null);
const startDate = ref(moment().startOf('day').format('YYYY-MM-DD'));
const endDate = ref(moment().startOf('day').format('YYYY-MM-DD'));
const search = ref('');
const data = ref([]);
const error = ref(null);
const loading = ref(true);
const menu = ref(false);
const sheet = ref(false);
const selectedClient = ref(null);
const tabview = ref('stocklist');
//coloumn visibility
const showMobileno = ref(true);
const showReceivedDate = ref(true);
const showReceivedTime = ref(true);
const showPlan = ref(true);
const showAction = ref(true);

const originalHeaders = [
  { title: 'ROW ID', value: 'rowId', sortable: true },
  { title: 'CLIENT CODE', value: 'ClientId', sortable: true },
  { title: 'CLIENT NAME', value: 'clientName', sortable: true },
  { title: 'MOBILE NUMBER', value: 'mobileNo', sortable: true },
  { title: 'RECEIVED DATE', value: 'receivedDate', sortable: true },
  { title: 'RECEIVED TIME', value: 'receivedTime', sortable: true },
  { title: 'PLAN', value: 'plan', sortable: true },
  { title: 'ACTION', value: 'action' },
];

const computedHeaders = computed(() => {
  return originalHeaders.filter(header => {
    if (header.value === 'mobileNo') return showMobileno.value;
    if (header.value === 'receivedDate') return showReceivedDate.value;
    if (header.value === 'receivedTime') return showReceivedTime.value;
    if (header.value === 'plan') return showPlan.value;
    if (header.value === 'action') return showAction.value;
    return true;
  });
});

onMounted(() => {
  if (dateInput.value) {
    $(dateInput.value).daterangepicker({
      timePicker: false,
      startDate: moment().startOf('day'),
      endDate: moment().startOf('day'),
      locale: {
        format: 'DD/MM/YYYY',
      },
    }, function(start, end) {
      startDate.value = start.format('YYYY-MM-DD');
      endDate.value = end.format('YYYY-MM-DD');
    });
  }

  // Load initial data on mount
  fetchData();
});

const fetchData = async () => {
  loading.value = true;
  error.value = null;
  const from = startDate.value;
  const to = endDate.value;
  const apiurl = 'https://g1.gwcindia.in/powerstocks/powerStocksView-v2.php';
  const formData = new FormData();
  formData.append('from', from);
  formData.append('to', to);

  try {
    const response = await fetch(apiurl, {
      method: 'POST',
      body: formData
    });

    if (!response.ok) {
      throw new Error('Network response was not ok');
    }

    const result = await response.json();
    data.value = result; // Set the fetched data

  } catch (err) {
    error.value = err.message;
  } finally {
    loading.value = false;
  }
};

const filteredProducts = computed(() => {
  const query = search.value.toLowerCase();
  return data.value.filter(item => {
    return (
      item.rowId.toLowerCase().includes(query) ||
      item.ClientId.toLowerCase().includes(query) ||
      item.clientName.toLowerCase().includes(query) ||
      item.mobileNo.toLowerCase().includes(query) ||
      item.receivedDate.toLowerCase().includes(query) ||
      item.receivedTime.toLowerCase().includes(query) ||
      item.plan.toLowerCase().includes(query)
    );
  });
});

const toggleColumn = (column) => {
  if (column === 'mobileNo') {
    showMobileno.value = !showMobileno.value;
  } else if (column === 'receivedDate') {
    showReceivedDate.value = !showReceivedDate.value;
  } else if (column === 'receivedTime') {
    showReceivedTime.value = !showReceivedTime.value;
  } else if (column === 'plan') {
    showPlan.value = !showPlan.value;
  } else if (column === 'action') {
    showAction.value = !showAction.value;
  }
};


// Export data to CSV
const exportDataToCsv = () => {
  const headers = ['Row ID', 'Client ID', 'Client Name', 'Mobile Number', 'Received Date', 'Received Time', 'Plan', 'Action'];
  const rows = filteredProducts.value.map(item => [
    item.rowId,
    item.ClientId,
    item.clientName,
    showMobileno.value ? item.mobileNo : '',
    showReceivedDate.value ? item.receivedDate : '',
    showReceivedTime.value ? item.receivedTime : '',
    showPlan.value ? item.plan : '',
    showAction.value ? 'View' : '' // Assuming 'View' as a placeholder for the action column
  ]);

  let csvContent = "data:text/csv;charset=utf-8," 
    + headers.join(",") + "\n" 
    + rows.map(e => e.join(",")).join("\n");

  const encodedUri = encodeURI(csvContent);
  const link = document.createElement("a");
  link.setAttribute("href", encodedUri);
  link.setAttribute("download", "data.csv");
  document.body.appendChild(link);
  link.click();
};


// Request post client code
const client_status_detail = ref([]);
const error_v = ref(null);
const loading_v = ref(true);
const viewDetails = async (item) => {
  sheet.value = true;
  selectedClient.value = item;

  const apiurl = 'https://g1.gwcindia.in/powerstocks/ps-sample-v2.php';
  const formData = new FormData();
  formData.append('clientCode', item.ClientId);
  formData.append('requestId', item.bulkReqId);
  
  try {
    const response_v = await fetch(apiurl, {
      method: 'POST',
      body: formData
    });
    
    if (!response_v.ok) {
      throw new Error('Network response was not ok');
    }
    
    // Assuming client_status_detail is used somewhere else for additional details
    client_status_detail.value = await response_v.json();
  } catch (err) {
    error_v.value = err.message;
  } finally {
    loading_v.value = false;
  }

 

};
const stockList = computed(() => client_status_detail.value.stockList || []);
const recommendation_notify = computed(() => client_status_detail.value.recommentation_notify || {});
const confirm_status = computed(() => client_status_detail.value.confirm_status || {});


const closeDetails = () => {
  sheet.value = false;
};
</script>

<style>
/* maintab section */
.tab-headers {
  display: flex;
  justify-content: space-around;
  cursor: pointer;
  margin-block-end: 20px;

}

.tab-headers div {
  flex: 1;
  border-radius: 5px;
  padding-block: 10px;
  padding-inline: 20px;
  text-align: center;
}

.tab-headers div.active {
  background-color:  #303964;
  color: white;
  font-weight: bold;
  
}

.tab-sections > div {
  display: none;
}

.tab-sections > div[tap] {
  display: block;
  border-radius: 5px;
}

/* view inside tab styling */
.tabs {
  display: flex;

}

.tabs button {
  border: none;
  background: none;
  cursor: pointer;
  font-size: 16px;
  outline: none;
  padding-block: 5px;
  padding-inline: 10px;
}

.tabs button.active {
  border-block-end: 3px solid #303964;

}

.tab-contentmain {
  padding: 20px;
  margin-block-start: 10px;
}

/* canva sheet */
.right-sheet {
  position: fixed;
  z-index: 1003;
  block-size: 100%;
  box-shadow: -2px 0 10px rgba(0, 0, 0, 10%);
  inline-size: 70%;
  inset-block-start: 0;
  inset-inline-end: 0;
  overflow-y: scroll;
  transform: translateX(100%);
  transition: transform 0.3s ease-in-out;
}

.right-sheet--active {
  transform: translateX(0);
}

.overlay {
  position: fixed;
  z-index: 1002;
  background-color: rgba(0, 0, 0, 50%);
  block-size: 100%;
  inline-size: 100%;
  inset-block-start: 0;
  inset-inline-start: 0;
}



</style>
