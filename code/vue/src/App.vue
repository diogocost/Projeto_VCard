<script setup>
import axios from 'axios'
import { inject, onMounted, ref } from 'vue'
import { useRouter, RouterLink, RouterView } from 'vue-router'
import { useToast } from "vue-toastification"
import { useUserStore } from './stores/user.js'
import { useTransactionsStore } from './stores/transactions'

const userStore = useUserStore()
const transactionsStore = useTransactionsStore()

const router = useRouter()
const toast = useToast()
const socket = inject("socket")


const logout = async () => {
  if (await userStore.logout()) {
    toast.success('User has logged out of the application.')
    router.push({ name: 'home' })
  } else {
    toast.error('There was a problem logging out of the application!')
  }
}

const clickMenuOption = () => {
  const domReference = document.getElementById('buttonSidebarExpandId')
  if (domReference) {
    if (window.getComputedStyle(domReference).display !== "none") {
      domReference.click()
    }
  }
}

onMounted(() => {
  socket.on('newTransaction', (transaction) => {
    toast.success(`Received new transaction #${transaction.id}, Amount : ${transaction.value}€!`)
    transactionsStore.loadTransactions()
  })
  socket.on('blockedUser', (user) => {
    userStore.clearUser()
    toast.error(`Your account has been blocked!`)
  })
  socket.on('deletedUser', (user) => {
    userStore.clearUser()
    router.push({ name: 'home' })
    toast.success(`Your account has been deleted!`)
  })
});
</script>


<template>
  <nav class="navbar navbar-expand-md navbar-dark bg-dark sticky-top flex-md-nowrap p-0 shadow">
    <div class="container-fluid">
      <router-link class="navbar-brand col-md-3 col-lg-2 me-0 px-3" :to="{ name: 'home' }" @click="clickMenuOption">
        <img src="@/assets/logo.svg" alt="" width="30" height="24" class="d-inline-block align-text-top">
        Vcard
      </router-link>
      <button id="buttonSidebarExpandId" class="navbar-toggler" type="button" data-bs-toggle="collapse"
        data-bs-target="#sidebarMenu" aria-controls="sidebarMenu" aria-expanded="false" aria-label="Toggle navigation">
        <span class="navbar-toggler-icon"></span>
      </button>

      <div class="collapse navbar-collapse justify-content-end">
        <ul class="navbar-nav">
          <li class="nav-item" v-show="!userStore.user"> <!-- v-show="!userStore.user" -->
            <router-link class="nav-link" :class="{ active: $route.name === 'NewVcard' }" :to="{ name: 'NewVcard' }"
              @click="clickMenuOption">
              <i class="bi bi-person-check-fill"></i>
              Register
            </router-link>
          </li>
          <li class="nav-item" v-show="!userStore.user"> <!-- v-show="!userStore.user" -->
            <router-link class="nav-link" :class="{ active: $route.name === 'Login' }" :to="{ name: 'Login' }"
              @click="clickMenuOption"> <!-- :class="{ active: $route.name === 'Login' }" -->
              <i class="bi bi-box-arrow-in-right"></i>
              Login
            </router-link>
          </li>
          <li class="nav-item dropdown" v-show="userStore.user"> <!-- v-show="userStore.user" -->
            <a class="nav-link dropdown-toggle" href="#" id="navbarDropdownMenuLink" role="button"
              data-bs-toggle="dropdown" aria-expanded="false">
              <img :src="userStore.userPhotoUrl" class="rounded-circle z-depth-0 avatar-img" alt="avatar image">
              <span class="avatar-text">{{ userStore.userName }}</span>
            </a>
            <ul class="dropdown-menu dropdown-menu-dark dropdown-menu-end" aria-labelledby="navbarDropdownMenuLink">
              <li>
                <router-link class="dropdown-item"
                  :class="{ active: $route.name == 'User' && $route.params.id == userStore.userId }"
                  :to="{ name: 'User' }" @click="clickMenuOption">
                  <!-- userStore.userId -->
                  <i class="bi bi-person-square"></i>
                  Profile
                </router-link>
              </li>
              <li>
                <router-link class="dropdown-item" :class="{ active: $route.name === 'ChangePassword' }"
                  :to="{ name: 'ChangePassword' }" @click="clickMenuOption"> <!--  -->
                  <i class="bi bi-key-fill"></i>
                  Change password
                </router-link>
              </li>
              <li v-if="!userStore.userIsAdmin">
                <router-link class="dropdown-item" :class="{ active: $route.name === 'ChangeConfirmationCode' }"
                  :to="{ name: 'ChangeConfirmationCode' }" @click="clickMenuOption"> <!--  -->
                  <i class="bi bi-phone"></i>
                  Change Confirmation Code
                </router-link>
              </li>
              <li>
                <hr class="dropdown-divider">
              </li>
              <li>
                <a class="dropdown-item" @click.prevent="logout">
                  <i class="bi bi-arrow-right"></i>Logout
                </a>
              </li>
            </ul>
          </li>
        </ul>
      </div>
    </div>
  </nav>
  <div class="container-fluid">
    <div class="row">
      <nav id="sidebarMenu" class="col-md-3 col-lg-2 d-md-block bg-light sidebar collapse">
        <div class="position-sticky pt-3">
          <ul class="nav flex-column"> <!-- v-if="userStore.user" -->
            <li class="nav-item">
              <router-link class="nav-link" :class="{ active: $route.name === 'dashboard' }" :to="{ name: 'dashboard' }"
                v-if="userStore.user" @click="clickMenuOption"> <!--  -->
                <i class="bi bi-house"></i>
                Dashboard
              </router-link>
            </li>
            <li class="nav-item d-flex justify-content-between align-items-center pe-3"
              v-if="!userStore.userIsAdmin && userStore.user">
              <router-link class="nav-link w-100 me-3" :class="{ active: $route.name === 'Transactions' }"
                :to="{ name: 'Transactions' }" @click="clickMenuOption"> <!--  -->
                <i class="bi bi-list-check"></i>
                Transactions
              </router-link>
              <router-link class="link-secondary" :to="{ name: 'NewTransaction' }" aria-label="Add a new transaction"
                @click="clickMenuOption"> <!--  -->
                <i class="bi bi-xs bi-plus-circle"></i>
              </router-link>
            </li>
            <li class="nav-item" v-if="!userStore.userIsAdmin && userStore.user">
              <router-link class="nav-link" :class="{ active: $route.name === 'Categories' }" :to="{ name: 'Categories' }"
                @click="clickMenuOption"> <!--  -->
                <i class="bi bi-files"></i>
                Categories
              </router-link>
            </li>
            <li class="nav-item" v-if="userStore.userIsAdmin">
              <router-link class="nav-link" :class="{ active: $route.name === 'Administrators' }"
                :to="{ name: 'Administrators' }" @click="clickMenuOption"> <!--  -->
                <i class="bi bi-files"></i>
                ADM Manage Administrators
              </router-link>
            </li>
            <li class="nav-item" v-if="userStore.userIsAdmin">
              <router-link class="nav-link" :class="{ active: $route.name === 'Vcards' }" :to="{ name: 'Vcards' }"
                @click="clickMenuOption"> <!--  -->
                <i class="bi bi-files"></i>
                ADM Manage Vcards
              </router-link>
            </li>
            <li class="nav-item" v-if="userStore.userIsAdmin">
              <router-link class="nav-link" :class="{ active: $route.name === 'DefaultCategories' }"
                :to="{ name: 'DefaultCategories' }" @click="clickMenuOption"> <!--  -->
                <i class="bi bi-files"></i>
                ADM Default Categories List
              </router-link>
            </li>
          </ul>

          <!-- HERE -->

        </div>
      </nav>

      <main class="col-md-9 ms-sm-auto col-lg-10 px-md-4">
        <router-view></router-view>
      </main>
    </div>
  </div>
</template>

<style>
@import "./assets/dashboard.css";

.avatar-img {
  margin: -1.2rem 0.8rem -2rem 0.8rem;
  width: 3.3rem;
  height: 3.3rem;
}

.avatar-text {
  line-height: 2.2rem;
  margin: 1rem 0.5rem -2rem 0;
  padding-top: 1rem;
}

.dropdown-item {
  font-size: 0.875rem;
}

.btn:focus {
  outline: none;
  box-shadow: none;
}

#sidebarMenu {
  overflow-y: auto;
}
</style>

