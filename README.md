        # [Tugas 9 - Pertemuan 10] 

**Nama** : Yacobus Daeli

**NIM** : H1D022024

**Shift** : E

Menjelaskan autentikasi login, hingga aplikasi mendapatkan username dan profil kita dari akun google.
Bonus : tampilkan foto profil di profil page

# Login Page 


![Tampilan Login](https://github.com/user-attachments/assets/5fc676ab-5060-4492-8c58-fa1cc3185ae0)


Di atas adalah tampilan pertama kalinya ketika kita membuka programn, untuk bisa akses ke dalam aplikasi kita akan melakukan login dengan cara memasukkan akun google dengan cara memencet button 'SIGN IN WITH GOOGLE'

**Source Code Tampilan Login (LoginPage.vue)**

```javascript
<ion-page>
        <ion-content :fullscreen="true">
            <div id="container">
                <!-- Title -->
                <ion-text style="margin-bottom: 20px; text-align: center;">
                    <h1>Praktikum Pemrograman Mobile</h1>
                </ion-text>

                <!-- Button Sign In -->
                <ion-button @click="login" color="light">
                    <ion-icon slot="start" :icon="logoGoogle"></ion-icon>
                    <ion-label>Sign In with Google</ion-label>
                </ion-button>
            </div>
        </ion-content>
    </ion-page>
```

Kode diatas adalah kode untuk menunjukkan teks dan button yang ada digambar tampilan login.

## Autentikasi Login Page

![Login with Google](https://github.com/user-attachments/assets/83ecae2c-40aa-4892-a59e-2b268acb67c4)

**Source Code Autentikasi Google** 

```javascript
<script setup lang="ts">
import { IonContent, IonPage, IonButton, IonIcon, IonText, IonLabel } from '@ionic/vue';
import { logoGoogle } from 'ionicons/icons';
import { useAuthStore } from '@/stores/auth';

const authStore = useAuthStore();

const login = async () => {
    await authStore.loginWithGoogle();
};
</script>
```

Untuk *imports*, kode diatas adalah untuk mengimpor beberapa komponen dari Ionic seperti 
 1. `IonContent, IonPage, IonButton, IonIcon, IonText, dan IonLabel` yang digunakan untuk membangun tampilan halaman.
 2. `logoGoogle } from 'ionicons/icons';`  , digunakan untuk Mengimpor logoGoogle dari ionicons/icons untuk menampilkan ikon Google dalam UI.
 3. `{ useAuthStore } from '@/stores/auth';`, digunakan untuk Mengimpor useAuthStore dari file store autentikasi (auth) yang telah didefinisikan sebelumnya. Store ini berisi fungsi-fungsi yang menangani autentikasi pengguna.

`const authStore = useAuthStore();` -> untuk kode ini menginisialisasi authStore dengan memanggil useAuthStore(). Ini memungkinkan penggunaan fungsi dan data yang ada dalam store auth di komponen ini.

`const login = async () => {
    await authStore.loginWithGoogle();`

Untuk kode ini digunakan untuk memanggil `authStore.loginWithGoogle()` untuk menjalankan proses login dengan Google. Ini menginisialisasi proses autentikasi yang telah didefinisikan di `auth.ts.`

Sebelum ini adanya pemanggilan instance diatas kita harus membuat file store autentikasi (auth) terlebih dahulu pada file `auth.ts`. sebelum itu kita
Buat sebuah folder `stores` di dalam folder `src` . Kemudian buat sebuah file `auth.ts` di dalam folder `stores`.

Berikutnya jalankan perintah `npm i --save @codetrix-studio/capacitor-google-auth`

**Source code Auth.ts**
```typescript
// file untuk autentikasi
// src/stores/auth.ts
import { defineStore } from 'pinia';
import { ref, computed } from 'vue';
import router from '@/router';
import { auth } from '@/utils/firebase';
import { GoogleAuthProvider, onAuthStateChanged, signInWithCredential, signOut, User } from 'firebase/auth';
import { GoogleAuth } from '@codetrix-studio/capacitor-google-auth';
import { alertController } from '@ionic/vue';

export const useAuthStore = defineStore('auth', () => {
    // Variabel User
    const user = ref<User | null>(null);

    // Variabel isAuth mengembalikan true or false
    // Cek jika user sudah login atau belum
    const isAuth = computed(() => user.value !== null);

    // Sign In with Google
    const loginWithGoogle = async () => {
        try {
            await GoogleAuth.initialize({
                clientId: '646929090731-kj8dalq5npha4hkvvum8bkjgf67ro7c7.apps.googleusercontent.com',
                scopes: ['profile', 'email'],
                grantOfflineAccess: true,
            });

            const googleUser = await GoogleAuth.signIn();

            const idToken = googleUser.authentication.idToken;

            const credential = GoogleAuthProvider.credential(idToken);

            const result = await signInWithCredential(auth, credential);

            user.value = result.user;

            router.push("/home");
        } catch (error) {
            console.error("Google sign-in error:", error);
            
            const alert = await alertController.create({
                header: 'Login Gagal!',
                message: 'Terjadi kesalahan saat login dengan Google. Coba lagi.',
                buttons: ['OK'],
            });

            await alert.present();

            throw error;
        }
    };

    // Logout
    const logout = async () => {
        try {
            await signOut(auth);
            await GoogleAuth.signOut();
            user.value = null;
            router.replace("/login");
        } catch (error) {
            console.error("Sign-out error:", error);
            throw error;
        }
    };

    // Fungsi bawaan firebase/auth untuk menyimpan informasi autentikasi pengguna
    onAuthStateChanged(auth, (currentUser) => {
        user.value = currentUser;
    });

    return { user, isAuth, loginWithGoogle, logout };
});
```

**Penjelasan kode `auth.ts` :** 
### Imports

- `defineStore` dari Pinia digunakan untuk mendefinisikan store, yaitu objek yang menyimpan state atau data yang bisa diakses di seluruh aplikasi.
- `ref` dan `computed` dari Vue untuk mendefinisikan variabel reaktif dan nilai terkomputasi.
- `router` dari `@/router` untuk navigasi antar halaman di aplikasi.
- `auth` dari `@/utils/firebase` dan beberapa metode dari Firebase Authentication (`GoogleAuthProvider`, `onAuthStateChanged`, `signInWithCredential`, `signOut`, `User`) untuk autentikasi pengguna.
- `GoogleAuth` dari `@codetrix-studio/capacitor-google-auth` untuk autentikasi Google pada platform mobile/hybrid.
- `alertController` dari `@ionic/vue` untuk menampilkan alert dalam aplikasi berbasis Ionic.

### Store `useAuthStore`

- `const user = ref<User | null>(null);`  
  Variabel `user` ini menyimpan data pengguna yang sedang login atau `null` jika belum ada yang login.

- `const isAuth = computed(() => user.value !== null);`  
  Properti terkomputasi `isAuth` mengembalikan `true` jika pengguna sudah login (`user` tidak `null`) dan `false` jika belum login.

### Fungsi `loginWithGoogle`

Fungsi ini menangani login menggunakan akun Google:
1. `GoogleAuth.initialize` untuk mengatur konfigurasi autentikasi Google seperti `clientId`, `scopes`, dan `grantOfflineAccess`.
2. `GoogleAuth.signIn()` membuka layar login Google.
3. Mengambil `idToken` dari hasil login dan membuat `credential` menggunakan `GoogleAuthProvider.credential(idToken)`.
4. `signInWithCredential(auth, credential)` untuk login di Firebase menggunakan token dari Google.
5. `user.value = result.user;` menyimpan data pengguna yang berhasil login.
6. `router.push("/home");` mengarahkan pengguna ke halaman beranda (`/home`) setelah login.
7. Jika terjadi kesalahan saat login, `alertController.create` menampilkan alert berisi pesan kesalahan untuk memberi tahu pengguna.

### Fungsi `logout`

Fungsi ini menangani proses logout:
1. `signOut(auth);` untuk logout dari Firebase.
2. `GoogleAuth.signOut();` untuk logout dari akun Google.
3. `user.value = null;` mengosongkan data pengguna.
4. `router.replace("/login");` mengarahkan pengguna ke halaman login setelah logout.

## `onAuthStateChanged`

Menjalankan listener yang otomatis memperbarui data `user` setiap kali ada perubahan status autentikasi, seperti login atau logout.

Sebelum itu, kita harus integrasi firebase terlebih dahulu. Firebase itu platform pengembangan aplikasi yang dibuat oleh Google. Platform ini menyediakan berbagai layanan dan alat untuk membantu pengembang membangun, meningkatkan, dan mengelola aplikasi mereka secara efisien. yang kita gunakan disini adalah **Firebase Authentication**.

Buat sebuah folder bernama `utils` di dalam folder `src` . Kemudian buat sebuah file bernama `firebase.ts` di dalam folder `utils`. 

Sesuaikan `firebaseConfig` dengan konfigurasi yang sudah dibuat sebelumnya di firebase console.

`firebase.ts`

```typescript
// src/utils/firebase.ts
import { initializeApp } from "firebase/app";
import { getAuth, GoogleAuthProvider } from 'firebase/auth';

const firebaseConfig = {
    apiKey: "",
    authDomain: "",
    projectId: "",
    storageBucket: "",
    messagingSenderId: "",
    appId: "",
};

const firebase = initializeApp(firebaseConfig);
const auth = getAuth(firebase);
const googleProvider = new GoogleAuthProvider();

export { auth, googleProvider };
```

**Penjelasan firebase.ts**

### Imports

```typescript
import { initializeApp } from "firebase/app";
import { getAuth, GoogleAuthProvider } from 'firebase/auth';
```
- initializeApp: Mengimpor fungsi untuk menginisialisasi aplikasi Firebase dengan konfigurasi yang diberikan.
- getAuth: Mengimpor fungsi untuk mendapatkan instance dari layanan autentikasi Firebase.
- GoogleAuthProvider: Mengimpor objek yang digunakan untuk mengatur login dengan Google.


### Konfigurasi Firebase

```typescript
const firebaseConfig = {
    apiKey: "AIzaSyBWIA2dvvLGecUSIq7LO0V1nfaeY4JnfCo",
    authDomain: "vue-firebase-41eeb.firebaseapp.com",
    projectId: "vue-firebase-41eeb",
    storageBucket: "vue-firebase-41eeb.firebasestorage.app",
    messagingSenderId: "646929090731",
    appId: "1:646929090731:web:d52d2a1a48a2c2302b4fee",
    measurementId: "G-GHMHWYPHQG"
};
```

- firebaseConfig: Objek ini berisi konfigurasi yang diperlukan untuk menghubungkan aplikasi ke proyek Firebase Anda. Konfigurasi ini termasuk informasi seperti:
- apiKey: Kunci API untuk aplikasi Anda.
- authDomain: Domain autentikasi untuk aplikasi Firebase.
- projectId: ID proyek Firebase Anda.
- storageBucket: Alamat tempat penyimpanan file Firebase.
- messagingSenderId: ID pengirim untuk notifikasi Firebase.
- appId: ID aplikasi untuk aplikasi Firebase Anda.
- measurementId: ID untuk pengukuran dan analitik di Firebase.

### Inisialisasi Firebase
`const firebase = initializeApp(firebaseConfig);`

- initializeApp(firebaseConfig): Fungsi ini digunakan untuk menginisialisasi aplikasi Firebase dengan konfigurasi yang diberikan dalam objek firebaseConfig. Ini menghubungkan aplikasi ke proyek Firebase yang terdaftar.

### Inisialisasi Autentikasi dan Provider Google

```typescript
const auth = getAuth(firebase);
const googleProvider = new GoogleAuthProvider();
```

- getAuth(firebase): Mengambil instance dari layanan autentikasi Firebase yang telah diinisialisasi dengan konfigurasi aplikasi Firebase.
- googleProvider = new GoogleAuthProvider(): Membuat instance dari GoogleAuthProvider yang digunakan untuk mengatur autentikasi menggunakan akun Google.

### Ekspor auth dan googleProvider

`export { auth, googleProvider }; `

- export { auth, googleProvider }: Menyediakan auth dan googleProvider untuk diekspor sehingga bisa digunakan di bagian lain aplikasi, seperti untuk mengelola autentikasi pengguna dengan Google.

Untuk mendapatkan Client ID, kita harus mengunjungi website https://console.cloud.google.com/apis/dashboard dan menuju ke menu vue-firebase karna yang kita gunakan adalah vue. Pilih menu **Credentials** lalu copy **Client ID**
setelah itu kita bisa memasukkan **Client ID** yang sudah di dapat dan kita bisa copykan ke file firebase.ts diatas.

Cara program mendapatkan username dan profil yaitu program menggunakan Google Sign-In melalui Firebase dan GoogleAuth dari @codetrix-studio/capacitor-google-auth untuk mendapatkan informasi pengguna setelah login dengan akun Google.

### Cara program mendapatkan username dan profil
**1. Login Dengan Google**

`const googleUser = await GoogleAuth.signIn();`
Pada fungsi loginWithGoogle di file auth.ts, proses login dengan Google dilakukan dengan memanfaatkan library GoogleAuth dari @codetrix-studio/capacitor-google-auth.

- GoogleAuth.signIn(): Fungsi ini membuka dialog login Google pada perangkat dan meminta pengguna untuk memberikan izin untuk mengakses data profil dan email mereka.
- Jika login berhasil, googleUser akan berisi informasi pengguna dari akun Google yang baru saja digunakan untuk login.

**2. Mengambil ID Token dan Membuat Credential**

Setelah login berhasil, ID Token dari pengguna diambil untuk digunakan dalam autentikasi Firebase.

```typescript
const idToken = googleUser.authentication.idToken;
const credential = GoogleAuthProvider.credential(idToken);

```

- googleUser.authentication.idToken: ID token ini adalah token yang dapat digunakan untuk mengautentikasi pengguna di Firebase dan digunakan untuk mengidentifikasi pengguna Google yang telah login.
- GoogleAuthProvider.credential(idToken): Membuat credential Firebase menggunakan ID token dari Google, yang akan digunakan untuk melakukan login ke Firebase.


**3. Login dengan Firebase menggunakan Credential Google**

Setelah memiliki credential, program melanjutkan untuk login ke Firebase dengan menggunakan signInWithCredential.

`const result = await signInWithCredential(auth, credential);`

signInWithCredential(auth, credential): Fungsi ini melakukan login ke Firebase menggunakan credential yang didapatkan dari login Google. Jika login berhasil, Firebase akan memberikan objek pengguna.

**4. Mendapatkan Data Pengguna**

Setelah login berhasil dengan Firebase, data pengguna akan disimpan di variabel user. Data ini akan berisi informasi tentang pengguna yang baru saja login.
`user.value = result.user;`

- result.user: Objek user ini berisi informasi lengkap tentang pengguna, termasuk username dan profil mereka.

**5. Mengakses Username dan Profil**

Setelah pengguna berhasil login, Kita dapat mengakses informasi profil mereka seperti nama, email, foto profil, dan lainnya melalui objek user yang ada di store auth.

Contoh : 
`const username = user.value.displayName;  // Mengambil nama pengguna
const profilePic = user.value.photoURL;  // Mengambil URL foto profil pengguna
`

Setelah itu kita bisa membuat `LoginPage.vue`, `HomePage.vue`, dan `ProfilePage.vue`. untuk `LoginPage.vue` kita sudah menjelaskan semua codenya diatas. 


# HomePage.vue

![Tampilan Home](https://github.com/user-attachments/assets/08c692af-c0a1-46ec-af14-6c68c70edce8)

## Source Code HomePage.vue
```typescript
<template>
  <ion-page>
    <ion-header :translucent="true">
      <ion-toolbar>
        <ion-title>Home</ion-title>
      </ion-toolbar>
    </ion-header>

    <ion-content :fullscreen="true">
      <div>
      </div>
      <TabsMenu />
    </ion-content>

  </ion-page>
</template>

<script setup lang="ts">
  import {
    IonContent,
    IonHeader,
    IonPage,
    IonTitle,
    IonToolbar
  } from '@ionic/vue';
  import TabsMenu from '@/components/TabsMenu.vue';
</script>
```
File HomePage.vue ini mendefinisikan sebuah halaman berjudul "Home" dengan layout yang mencakup header di bagian atas dan konten yang dapat disesuaikan di bagian bawah. Halaman ini juga menggunakan komponen TabsMenu untuk menyediakan navigasi tab bagi pengguna. Semua elemen UI utama, seperti header dan konten, diambil dari komponen-komponen yang disediakan oleh Ionic.


# ProfilePage.vue (Bonus dengan menampilkan foto profil di page ProfilePage.vue)


![Tampilan Profile](https://github.com/user-attachments/assets/23fd0e2c-b76e-428a-836f-b009e8dbd689)


## Source code ProfilePage.vue

```typescript
<template>
    <ion-page>
        <ion-header :translucent="true">
            <ion-toolbar>
                <ion-title>Profile</ion-title>

                <!-- Logout Button -->
                <ion-button slot="end" fill="clear" @click="logout" style="--color: gray;">
                    <ion-icon slot="end" :icon="exit"></ion-icon>
                    <ion-label>Logout</ion-label>
                </ion-button>
            </ion-toolbar>
        </ion-header>

        <ion-content :fullscreen="true">
            <!-- Avatar -->
            <div id="avatar-container">
                <ion-avatar>
                    <img alt="Avatar" :src="userPhoto" @error="handleImageError" />
                </ion-avatar>
            </div>

            <!-- Data Profile -->
            <ion-list>
                <ion-item>
                    <ion-input label="Nama" :value="user?.displayName" :readonly="true"></ion-input>
                </ion-item>

                <ion-item>
                    <ion-input label="Email" :value="user?.email" :readonly="true"></ion-input>
                </ion-item>
            </ion-list>

            <!-- Tabs Menu -->
            <TabsMenu />
        </ion-content>

    </ion-page>
</template>

<script setup lang="ts">
import { IonContent, IonHeader, IonPage, IonTitle, IonToolbar, IonInput, IonItem, IonList, IonLabel, IonIcon, IonButton, IonAvatar } from '@ionic/vue';
import { exit } from 'ionicons/icons';
import { computed, ref } from 'vue';
import TabsMenu from '@/components/TabsMenu.vue';
import { useAuthStore } from '@/stores/auth';

const authStore = useAuthStore();
const user = computed(() => authStore.user);

const logout = () => {
    authStore.logout();
};

const userPhoto = ref(user.value?.photoURL || 'https://ionicframework.com/docs/img/demos/avatar.svg');

function handleImageError() {
    userPhoto.value = 'https://ionicframework.com/docs/img/demos/avatar.svg';
}
</script>

<style scoped>
#avatar-container {
    display: flex;
    justify-content: center;
    align-items: center;
    margin: 20px 0;
}

#avatar-icon {
    width: 80px;
    height: 80px;
}
</style>
```

File ProfilePage.vue ini mendefinisikan halaman profil pengguna di aplikasi. Halaman ini menampilkan foto profil pengguna, nama, dan email. Pengguna dapat melakukan logout melalui tombol di toolbar, yang akan memanggil fungsi logout dari store auth. Komponen TabsMenu digunakan untuk menyediakan navigasi antar tab dalam aplikasi. Gambar profil diatur agar jika ada kesalahan saat memuat gambar, akan menggunakan gambar default.

**Script Section**

- **`useAuthStore`**: Mengimpor store `auth` untuk mengakses informasi pengguna, seperti data profil pengguna yang sedang login.

- **`const user = computed(() => authStore.user);`**: Menggunakan fungsi `computed` untuk mendapatkan data pengguna yang sedang login dari store `auth`. Ini akan memberikan reaktifitas, sehingga setiap kali data pengguna berubah, nilai ini akan otomatis diperbarui.

- **`logout`**: Fungsi ini digunakan untuk keluar dari aplikasi dengan memanggil metode `logout` dari store `auth`, yang akan menghapus sesi pengguna yang sedang aktif.

- **`const userPhoto`**: Menginisialisasi variabel `userPhoto` dengan URL foto profil pengguna yang diambil dari properti `photoURL` dalam data pengguna. Jika pengguna tidak memiliki foto profil, maka gambar default akan digunakan sebagai fallback.

- **`handleImageError`**: Fungsi ini menangani kesalahan yang mungkin terjadi saat memuat gambar avatar. Jika gambar gagal dimuat, maka gambar default akan ditampilkan.


Terahkir kita harus mengkonfigurasi router yang berada src/router/index.ts

# Index.ts

## Source code index.ts

```javascript
import { createRouter, createWebHistory } from '@ionic/vue-router';
import { RouteRecordRaw } from 'vue-router';
import HomePage from '@/views/HomePage.vue';
import LoginPage from '@/views/LoginPage.vue';
import { useAuthStore } from '@/stores/auth';
import ProfilePage from '@/views/ProfilePage.vue';
import { onAuthStateChanged } from 'firebase/auth';
import { auth } from '@/utils/firebase';

const routes: Array<RouteRecordRaw> = [
  {
    path: '/',
    redirect: '/login',
  },
  {
    path: '/login',
    name: 'login',
    component: LoginPage,
    meta: {
      isAuth: false,
    },
  },
  {
    path: '/home',
    name: 'home',
    component: HomePage,
    meta: {
      isAuth: true,
    },
  },
  {
    path: '/profile',
    name: 'profile',
    component: ProfilePage,
    meta: {
      isAuth: true,
    },
  },
];

const router = createRouter({
  history: createWebHistory(import.meta.env.BASE_URL),
  routes,
});

router.beforeEach(async (to, from, next) => {
  const authStore = useAuthStore();

  if (authStore.user === null) {
    await new Promise<void>((resolve) => {
      const unsubscribe = onAuthStateChanged(auth, () => {
        resolve();
        unsubscribe();
      });
    });
  }

  if (to.path === '/login' && authStore.isAuth) {
    next('/home');
  } else if (to.meta.isAuth && !authStore.isAuth) {
    next('/login');
  } else {
    next();
  }
});

export default router;
```

### Imports

```typescript

import { createRouter, createWebHistory } from '@ionic/vue-router';
import { RouteRecordRaw } from 'vue-router';
import HomePage from '@/views/HomePage.vue';
import LoginPage from '@/views/LoginPage.vue';
import { useAuthStore } from '@/stores/auth';
import ProfilePage from '@/views/ProfilePage.vue';
import { onAuthStateChanged } from 'firebase/auth';
import { auth } from '@/utils/firebase';
```

- createRouter dan createWebHistory: Fungsi dari @ionic/vue-router untuk membuat router di aplikasi Ionic yang menggunakan mode sejarah (history mode).
- RouteRecordRaw: Tipe data dari Vue Router yang digunakan untuk mendefinisikan rute.
- HomePage, LoginPage, ProfilePage: Komponen halaman yang diimpor untuk ditampilkan sesuai dengan rute yang dituju.
- useAuthStore: Mengimpor store auth untuk mengakses status autentikasi pengguna.
- onAuthStateChanged dan auth: Digunakan untuk memantau status autentikasi pengguna menggunakan Firebase, untuk memverifikasi apakah pengguna sudah login atau belum.


### Definisi Routes

```typescript

const routes: Array<RouteRecordRaw> = [
  {
    path: '/',
    redirect: '/login',
  },
  {
    path: '/login',
    name: 'login',
    component: LoginPage,
    meta: {
      isAuth: false,
    },
  },
  {
    path: '/home',
    name: 'home',
    component: HomePage,
    meta: {
      isAuth: true,
    },
  },
  {
    path: '/profile',
    name: 'profile',
    component: ProfilePage,
    meta: {
      isAuth: true,
    },
  },
];
```

- routes: Menyimpan daftar rute yang tersedia di aplikasi. Setiap objek dalam array mendefinisikan sebuah rute.
- Rute dengan path '/' mengarah ke /login jika pengguna tidak memiliki rute yang sesuai.
- Rute untuk /login digunakan untuk halaman login, dan memiliki metadata isAuth: false, artinya hanya dapat diakses jika pengguna belum login.
- Rute /home dan /profile hanya dapat diakses jika pengguna sudah terautentikasi, yang ditandai dengan metadata isAuth: true.

### Membuat Router 

`const router = createRouter({
  history: createWebHistory(import.meta.env.BASE_URL),
  routes,
});
`

- createRouter: Membuat instance router dengan konfigurasi sejarah berbasis URL (history mode) dan daftar rute yang telah didefinisikan.
- import.meta.env.BASE_URL: Menggunakan URL dasar yang diambil dari konfigurasi build aplikasi.

### Middleware untuk Navigasi


```typescript
router.beforeEach(async (to, from, next) => {
  const authStore = useAuthStore();

  if (authStore.user === null) {
    await new Promise<void>((resolve) => {
      const unsubscribe = onAuthStateChanged(auth, () => {
        resolve();
        unsubscribe();
      });
    });
  }

  if (to.path === '/login' && authStore.isAuth) {
    next('/home');
  } else if (to.meta.isAuth && !authStore.isAuth) {
    next('/login');
  } else {
    next();
  }
});
```

- router.beforeEach: Fungsi ini adalah navigasi guard yang dijalankan sebelum setiap perubahan rute.
- authStore.user: Mengecek apakah pengguna sudah login. Jika pengguna belum login (user === null), aplikasi menunggu status autentikasi dari Firebase dengan menggunakan onAuthStateChanged. Hal ini memastikan bahwa aplikasi menunggu informasi status autentikasi pengguna sebelum melanjutkan navigasi.
- next(): Fungsi yang digunakan untuk melanjutkan navigasi. Berdasarkan kondisi:
- Jika pengguna sudah login dan mencoba mengakses halaman login (to.path === '/login'), pengguna akan diarahkan ke halaman /home.
- Jika pengguna belum login dan mencoba mengakses halaman yang memerlukan autentikasi (to.meta.isAuth === true), mereka akan diarahkan ke halaman login (/login).
- Jika rute valid, navigasi akan dilanjutkan dengan next().

# [Tugas 10 - Pertemuan 11]
Pada pertemuan 11, disini melakukan implementasi CRUD, dengan program TO DO LIST sederhana.

# HomePage.vue
Berikut adalah tampilan utama untuk program sederhana to - do list.

![Tampilan Home](https://github.com/user-attachments/assets/03089ee6-a418-4117-80de-72dc9869398d)


Kode untuk tampilan HomePage.vue :

```vue

<template>
  <ion-page>
    <ion-header :translucent="true">
      <ion-toolbar>
        <ion-title>Home</ion-title>
      </ion-toolbar>
    </ion-header>

  <!-- modifikasi src/views/HomePage.vue pada bagian ion-content dalam template -->
<ion-content :fullscreen="true">
	<!-- komponen paling atas dari ion-content -->
	<ion-refresher slot="fixed" :pull-factor="0.5" :pull-min="100" :pull-max="200"
    @ionRefresh="handleRefresh($event)">
    <ion-refresher-content></ion-refresher-content>
  </ion-refresher>
  
  <!-- komponen utama di antara 2 komponen ini -->
  
  <!-- komponen paling bawah dari ion-content -->
  <ion-fab vertical="bottom" horizontal="end" slot="fixed">
    <ion-fab-button @click="isOpen = true;">
      <ion-icon :icon="add" size="large"></ion-icon>
    </ion-fab-button>
  </ion-fab>
  <InputModal v-model:isOpen="isOpen" v-model:editingId="editingId" :todo="todo" @submit="handleSubmit" />

  <!-- bagian refresher -->

<!-- active todos -->
<div class="scrollable-container">
  <ion-list>
    <ion-item-sliding v-for="todo in activeTodos" :key="todo.id" :ref="(el) => setItemRef(el, todo.id!)">
      <ion-item-options side="start" @ionSwipe="handleDelete(todo)">
        <ion-item-option color="danger" expandable @click="handleDelete(todo)">
          <ion-icon slot="icon-only" :icon="trash" size="large"></ion-icon>
        </ion-item-option>
      </ion-item-options>

      <ion-item>
        <ion-card>
          <ion-card-header>
            <ion-card-title class="ion-text-wrap limited-text">{{ todo.title }}</ion-card-title>
            <ion-card-subtitle class="limited-text">{{ todo.description }}</ion-card-subtitle>
          </ion-card-header>

          <ion-card-content>
            <ion-badge>{{ getRelativeTime(todo.updatedAt) }}</ion-badge>
          </ion-card-content>
        </ion-card>
      </ion-item>

      <ion-item-options side="end" @ionSwipe="handleStatus(todo)">
        <ion-item-option @click="handleEdit(todo)">
          <ion-icon slot="icon-only" :icon="create" size="large"></ion-icon>
        </ion-item-option>
        <ion-item-option color="success" expandable @click="handleStatus(todo)">
          <ion-icon slot="icon-only" :icon="checkmarkCircle" color="light" size="large"></ion-icon>
        </ion-item-option>
      </ion-item-options>
    </ion-item-sliding>
    <ion-item v-if="activeTodos.length === 0" class="ion-text-center">
      <ion-label>No active todos</ion-label>
    </ion-item>
  </ion-list>
</div>

<!-- completed todos -->
<ion-item class="accordion-container">
  <ion-accordion-group>
    <ion-accordion value="first">
      <ion-item slot="header" color="light">
        <ion-label class="ion-text-center">Completed</ion-label>
      </ion-item>
      <div slot="content" class="scrollable-container">
        <ion-list>
          <ion-item-sliding v-for="todo in completedTodos" :key="todo.id" :ref="(el) => setItemRef(el, todo.id!)">
            <ion-item-options side="start" @ionSwipe="handleDelete(todo)">
              <ion-item-option color="danger" expandable @click="handleDelete(todo)">
                <ion-icon slot="icon-only" :icon="trash" size="large"></ion-icon>
              </ion-item-option>
            </ion-item-options>

            <ion-item>
              <ion-card>
                <ion-card-header>
                  <ion-card-title class="ion-text-wrap limited-text">{{ todo.title }}</ion-card-title>
                  <ion-card-subtitle class="limited-text">{{ todo.description }}</ion-card-subtitle>
                </ion-card-header>

                <ion-card-content>
                  <ion-badge>{{ getRelativeTime(todo.updatedAt) }}</ion-badge>
                </ion-card-content>
              </ion-card>
            </ion-item>

            <ion-item-options side="end" @ionSwipe="handleStatus(todo)">
              <ion-item-option @click="handleEdit(todo)">
                <ion-icon slot="icon-only" :icon="create" size="large"></ion-icon>
              </ion-item-option>
              <ion-item-option color="warning" expandable @click="handleStatus(todo)">
                <ion-icon slot="icon-only" :icon="close" color="light" size="large"></ion-icon>
              </ion-item-option>
            </ion-item-options>
          </ion-item-sliding>
          <ion-item v-if="completedTodos.length === 0" class="ion-text-center">
            <ion-label>No completed todos</ion-label>
          </ion-item>
        </ion-list>
      </div>
    </ion-accordion>
  </ion-accordion-group>
</ion-item>

<!-- bagian tombol dan modal -->
</ion-content>

  </ion-page>
</template>


<!-- modifikasi src/views/HomePage.vue import semua komponen yang diperlukan -->
<script setup lang="ts">
import {
  IonContent,
  IonHeader,
  IonPage,
  IonTitle,
  IonToolbar,
  IonItem,
  IonItemOption,
  IonItemOptions,
  IonItemSliding,
  IonCard,
  IonCardHeader,
  IonCardTitle,
  IonCardSubtitle,
  IonCardContent,
  IonBadge,
  IonIcon,
  IonFab,
  IonFabButton,
  IonAccordion,
  IonAccordionGroup,
  IonLabel,
  IonList,
  loadingController,
  IonRefresher,
  IonRefresherContent,
  toastController
} from '@ionic/vue';
import {
  add,
  checkmarkCircle,
  close,
  create,
  trash,
  closeCircle,
  warningOutline
} from 'ionicons/icons';
import InputModal from '../components/InputModal.vue';
import { onMounted, ref, computed, onUnmounted } from 'vue';
import { firestoreService, type Todo } from '@/utils/firestore';
import { formatDistanceToNow } from 'date-fns';

// modifikasi src/views/HomePage.vue deklarasikan variabel yang akan digunakan
const isOpen = ref(false);
const editingId = ref<string | null>(null);
const todos = ref<Todo[]>([]);
const todo = ref<Omit<Todo, 'id' | 'createdAt' | 'updatedAt' | 'status'>>({
  title: '',
  description: '',
});
const activeTodos = computed(() => todos.value.filter(todo => !todo.status));
const completedTodos = computed(() => todos.value.filter(todo => todo.status));
const itemRefs = ref<Map<string, HTMLIonItemSlidingElement>>(new Map());
let intervalId: any;
const timeUpdateTrigger = ref(0);

// mendapatkan value dari item
const setItemRef = (el: any, id: string) => {
  if (el) {
    itemRefs.value.set(id, el.$el);
  }
};

// toast notifikasi
const showToast = async (message: string, color: string = 'success', icon: string = checkmarkCircle) => {
  const toast = await toastController.create({
    message,
    duration: 2000,
    color,
    position: 'top',
    icon
  });
  await toast.present();
};

// mendapatkan perbedaan waktu
const getRelativeTime = (date: any) => {
  timeUpdateTrigger.value;
  try {
    const jsDate = date?.toDate ? date.toDate() : new Date(date);
    return formatDistanceToNow(jsDate, { addSuffix: true });
  } catch (error) {
    return 'Invalid date';
  }
};

// load data
const loadTodos = async (isLoading = true) => {
  let loading;
  if (isLoading) {
    loading = await loadingController.create({
      message: 'Loading...'
    });
    await loading.present();
  }

  try {
    todos.value = await firestoreService.getTodos();
  } catch (error) {
    console.error(error);
  } finally {
    if (loading) {
      await loading.dismiss();
    }
  }
};

// dijalankan setiap halaman diload, load data dan set interval update 1 menit
onMounted(() => {
  loadTodos();
  intervalId = setInterval(() => {
    timeUpdateTrigger.value++;
  }, 60000);
});

// reset interval update
onUnmounted(() => {
  clearInterval(intervalId);
});

// handle swipe refresher
const handleRefresh = async (event: any) => {
  try {
    await loadTodos(false);
  } catch (error) {
    console.error('Error refreshing:', error);
  } finally {
    event.target.complete();
  }
};

// handle submit add/edit pada modal
const handleSubmit = async (todo: Omit<Todo, 'id' | 'createdAt' | 'updatedAt' | 'status'>) => {
  if (!todo.title) {
    await showToast('Title is required', 'warning', warningOutline);
    return;
  }
  try {
    if (editingId.value) {
      await firestoreService.updateTodo(editingId.value, todo as Todo);
      await showToast('Todo updated successfully', 'success', checkmarkCircle);
    } else {
      await firestoreService.addTodo(todo as Todo);
      await showToast('Todo added successfully', 'success', checkmarkCircle);
    }
    loadTodos();
  } catch (error) {
    await showToast('An error occurred', 'danger', closeCircle);
    console.error(error);
  } finally {
    editingId.value = null;
  }
};
// handle edit click
const handleEdit = async (editTodo: Todo) => {
  const slidingItem = itemRefs.value.get(editTodo.id!);
  await slidingItem?.close();

  editingId.value = editTodo.id!;
  todo.value = {
    title: editTodo.title,
    description: editTodo.description,
  }
  isOpen.value = true;
}

// handle delete click/swipe
const handleDelete = async (deleteTodo: Todo) => {
  try {
    await firestoreService.deleteTodo(deleteTodo.id!);
    await showToast('Todo deleted successfully', 'success', checkmarkCircle);
    loadTodos();
  } catch (error) {
    await showToast('Failed to delete todo', 'danger', closeCircle);
    console.error(error);
  }
};

// handle status click/swipe, mengubah status todo active (false)/completed (true)
const handleStatus = async (statusTodo: Todo) => {
  const slidingItem = itemRefs.value.get(statusTodo.id!);
  await slidingItem?.close();
  try {
    await firestoreService.updateStatus(statusTodo.id!, !statusTodo.status);
    await showToast(
      `Todo marked as ${!statusTodo.status ? 'completed' : 'active'}`,
      'success',
      checkmarkCircle
    );
    loadTodos();
  } catch (error) {
    await showToast('Failed to update status', 'danger', closeCircle);
    console.error(error);
  }
};


</script>
l
<!-- modifikasi src/views/HomePage.vue tambahkan keseluruhan style -->
<style scoped>
ion-card,
ion-accordion-group {
  width: 100%;
}

ion-fab {
  margin: 25px;
}

.limited-text {
  overflow: hidden;
  text-overflow: ellipsis;
  display: -webkit-box;
  -webkit-box-orient: vertical;
}

ion-card-title.limited-text {
  -webkit-line-clamp: 1;
  line-clamp: 1;
}

ion-card-subtitle.limited-text {
  -webkit-line-clamp: 2;
  line-clamp: 2;
}

.scrollable-container {
  max-height: 80vh;
  overflow-y: auto;
  overflow-x: hidden;
}

.accordion-container {
  --padding-start: 0;
  --inner-padding-end: 0;
}

ion-item {
  --padding-start: 0;
  --inner-padding-end: 0;
}

.scrollable-container::-webkit-scrollbar {
  width: 8px;
}

.scrollable-container::-webkit-scrollbar-track {
  background: #f1f1f1;
}

.scrollable-container::-webkit-scrollbar-thumb {
  background: #888;
  border-radius: 4px;
}

.scrollable-container::-webkit-scrollbar-thumb:hover {
  background: #555;
}
</style>

```

Penjelasan kode untuk halamana HomePage.vue : 

## Header 
```vue
<ion-header :translucent="true">
  <ion-toolbar>
    <ion-title>Home</ion-title>
  </ion-toolbar>
</ion-header>
```

- `ion-header`: Membuat header halaman.
- `ion-toolbar`: Toolbar di dalam header.
- `ion-title`: Menampilkan teks juqdul "Home".

## Refresher
```vue
<ion-refresher slot="fixed" :pull-factor="0.5" :pull-min="100" :pull-max="200"
  @ionRefresh="handleRefresh($event)">
  <ion-refresher-content></ion-refresher-content>
</ion-refresher>
```
Ion refresher digunakan untuk menyegarkan data dengan gesture menarik layar ke bawah.
`pull-factor`: Mengontrol sensitivitas tarik.
`pull-min & pull-max`: Menentukan jarak minimum/maksimum untuk memicu refresh.

`@ionRefresh` : Memanggil fungsi **handleRefresh** saat gesture selesai.

## Tombol Tambah (Floating Action Button)
```vue
<ion-fab vertical="bottom" horizontal="end" slot="fixed">
  <ion-fab-button @click="isOpen = true;">
    <ion-icon :icon="add" size="large"></ion-icon>
  </ion-fab-button>
</ion-fab>
<InputModal v-model:isOpen="isOpen" v-model:editingId="editingId" :todo="todo" @submit="handleSubmit" />
```

- `ion-fab`: Tombol melayang di sudut kanan bawah.
- `ion-fab-button`: Membuka modal input (mengubah isOpen ke true).
- `InputModal`: Modal untuk menambah atau mengedit todo.
- `v-model` : **isOpen** untuk mengontrol buka/tutup modal. **editingId** untuk menentukan ID todo yang diedit.
- `Event` : **@submit** memanggil **handleSubmit** saat data disubmit.

## Daftar list TODO yang aktif
```vue
<ion-list>
  <ion-item-sliding v-for="todo in activeTodos" :key="todo.id" :ref="(el) => setItemRef(el, todo.id!)">
    <!-- Opsi hapus -->
    <ion-item-options side="start" @ionSwipe="handleDelete(todo)">
      <ion-item-option color="danger" expandable @click="handleDelete(todo)">
        <ion-icon slot="icon-only" :icon="trash" size="large"></ion-icon>
      </ion-item-option>
    </ion-item-options>
    <!-- Detail Todo -->
    <ion-item>
      <ion-card>
        <ion-card-header>
          <ion-card-title class="ion-text-wrap limited-text">{{ todo.title }}</ion-card-title>
          <ion-card-subtitle class="limited-text">{{ todo.description }}</ion-card-subtitle>
        </ion-card-header>
        <ion-card-content>
          <ion-badge>{{ getRelativeTime(todo.updatedAt) }}</ion-badge>
        </ion-card-content>
      </ion-card>
    </ion-item>
    <!-- Opsi edit & ubah status -->
    <ion-item-options side="end" @ionSwipe="handleStatus(todo)">
      <ion-item-option @click="handleEdit(todo)">
        <ion-icon slot="icon-only" :icon="create" size="large"></ion-icon>
      </ion-item-option>
      <ion-item-option color="success" expandable @click="handleStatus(todo)">
        <ion-icon slot="icon-only" :icon="checkmarkCircle" color="light" size="large"></ion-icon>
      </ion-item-option>
    </ion-item-options>
  </ion-item-sliding>
  <ion-item v-if="activeTodos.length === 0" class="ion-text-center">
    <ion-label>No active todos</ion-label>
  </ion-item>
</ion-list>
```

- `v-for`: Mengulang daftar todo aktif.
- `ion-item-sliding`: Setiap todo dapat diswipe untuk aksi.
- `ion-item-options`:
- `side="start"`: Untuk hapus (swipe ke kiri).
- `side="end"`: Untuk edit atau ubah status (swipe ke kanan).
- `ion-item`: Menampilkan informasi todo dengan desain kartu.
- `ion-badge`: Menampilkan waktu relatif dari pembaruan terakhir.
- Jika tidak ada TODO / atau kosong, maka akan menampilkan pesan **"No Active todos"**


## List TODO Selesai

```vue
<ion-accordion-group>
  <ion-accordion value="first">
    <ion-item slot="header" color="light">
      <ion-label class="ion-text-center">Completed</ion-label>
    </ion-item>
    <div slot="content" class="scrollable-container">
      <ion-list>
        <!-- Mirip Todo Aktif -->
      </ion-list>
    </div>
  </ion-accordion>
</ion-accordion-group>
```
Sebelumnya code diatas mirip dengan daftar todo aktif, tetapi:
- Menggunakan `ion-accordion` untuk menyembunyikan daftar selesai.
- Label header adalah **"Completed"**.

## Bagian Script
1. State dan Komputasi
```javascript
const isOpen = ref(false);
const editingId = ref<string | null>(null);
const todos = ref<Todo[]>([]);
const activeTodos = computed(() => todos.value.filter(todo => !todo.status));
const completedTodos = computed(() => todos.value.filter(todo => todo.status));
```
- `isOpen`: Status buka/tutup modal.
- `ditingId`: ID todo yang diedit.
- `todos`: Semua data todo.
- `activeTodos` & `completedTodos`: Mengelompokkan todo berdasarkan status.

2. Fungsi Pendukung
```javascript
const showToast = async (message: string, color: string = 'success', icon: string = checkmarkCircle) => { ... };
const getRelativeTime = (date: any) => { ... };
```
- `showToast`: Menampilkan notifikasi.
- `getRelativeTime`: Menghitung waktu relatif dari tanggal.

3. Fungsi CRUD (Create, Read, Update, Delete)
```javascript
const loadTodos = async () => { ... };
const handleSubmit = async (todo: Omit<Todo, 'id'>) => { ... };
const handleEdit = async (todo: Todo) => { ... };
const handleDelete = async (todo: Todo) => { ... };
const handleStatus = async (todo: Todo) => { ... };
```
- `loadTodos`: Memuat data todo dari server.
- `handleSubmit`: Menambah atau mengedit todo.
- `handleEdit`: Membuka modal dengan data todo untuk diedit.
- `handleDelete`: Menghapus todo.
- `handleStatus`: Mengubah status todo (aktif/selesai).

4. Lifecycle Hooks
```javascript
onMounted(() => {
  loadTodos();
  intervalId = setInterval(() => {
    timeUpdateTrigger.value++;
  }, 60000);
});
onUnmounted(() => {
  clearInterval(intervalId);
});
```
- `onMounted`: Memuat data awal dan memperbarui waktu setiap 1 menit.
- `onUnmounted`: Membersihkan interval saat komponen dilepas.
