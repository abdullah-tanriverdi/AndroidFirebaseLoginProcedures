# Firebase Authentication Entegrasyonu

## **Firebase Projesi Oluşturma**

1. **Firebase Console’a Giriş Yapma**  
   [Firebase Console](https://console.firebase.google.com/) adresine gidiyoruz.

2. **Yeni Proje Oluşturma**  
   Firebase Console ana sayfasında, sağ üst köşede bulunan **"Add Project"** (Proje Ekle) butonuna tıklıyoruz. Ardından, karşımıza çıkan adımları takip ederek yeni bir proje oluşturuyoruz. Proje adı ve diğer gerekli bilgileri girerek işlemi tamamlıyoruz.

3. **Firebase Projesinin Hazırlanması**  
   Projemiz oluşturulduktan sonra, Firebase Console'da yeni projemiz aktif hale gelecektir.

4. **Firebase Authentication Özelliğini Aktif Hale Getirme**  
   Projemiz hazır olduktan sonra, sol menüde yer alan **Authentication** sekmesine tıklıyoruz. Ardından, **"Get Started"** (Başlayın) butonuna tıklayarak Authentication özelliğini aktifleştiriyoruz.

---

## **Android Studio Projesini Firebase Projesine Bağlama**

1. **Firebase Sekmesine Tıklama**  
   Android Studio içinde, üst menüde bulunan **Firebase** sekmesine tıklıyoruz.

2. **Authentication Sekmesine Erişim**  
   Sol menüde yer alan **Authentication** sekmesine tıklıyoruz. Bu, Firebase Authentication hizmetini proje ile entegre etmek için gerekli ayarları yapmamıza olanak sağlar.

3. **Custom Authentication Sistemi Seçimi**  
   Authentication sekmesinin altında bulunan alt seçeneklerden **Authenticate using a custom authentication system** kısmına tıklıyoruz.

4. **Firebase’e Bağlanma**  
   Karşımıza çıkan pencerede, **Connect to Firebase** butonuna tıklıyoruz. Bu adım, Android Studio projemizi Firebase projemize bağlamamıza olanak tanır.

5. **SDK’yı Projeye Ekleme**  
   Firebase ile bağlantı kurduktan sonra, **Add the Firebase Authentication SDK to your project** seçeneğini tıklıyoruz. Bu, gerekli Firebase Authentication bağımlılıklarını projemize ekleyecektir.

6. **Değişiklikleri Kabul Etme**  
   Bağımlılıkları ekledikten sonra, **Accept Changes** seçeneğine tıklayarak, gerekli yapılandırma ve dosya değişikliklerini onaylıyoruz.

7. **Firebase Console’a Giriş**  
   Firebase Console’a gidiyoruz ve proje üzerinde çalışmaya devam ediyoruz.

8. **Authentication Ayarlarını Yapma**  
   Firebase Console’da, sol menüden **Authentication** sekmesine tıklıyoruz ve ardından **Sign-in method** kısmına geliyoruz.

9. **E-posta ve Şifre ile Giriş Yapmayı Etkinleştirme**  
   **Sign-in method** altında **Email/Password** seçeneğini aktif hale getiriyoruz. Bu sayede kullanıcılar, e-posta ve şifre kullanarak uygulamaya giriş yapabilecekler.

---

## **Firebase Authentication Yapılandırması**

![Firebase Authentication]("Ekran görüntüsü 2024-11-19 133224.png")

### AuthViewModel Sınıfı

Firebase Authentication ile kullanıcı işlemlerini yönetmek için bir `AuthViewModel` sınıfı oluşturuyoruz. Bu sınıf, kullanıcıların kayıt olma, giriş yapma ve şifre sıfırlama işlemlerini yönetir.

Örnek bir kullanıcı kaydı fonksiyonu:

```
fun registerUser(email: String, password: String) {
    auth.createUserWithEmailAndPassword(email, password)
        .addOnCompleteListener { task ->
            if (task.isSuccessful) {
                val user = auth.currentUser
                user?.sendEmailVerification()
                _authState.value = AuthState.Success
            } else {
                _authState.value = AuthState.Error(task.exception?.message ?: "Registration failed")
            }
        }
}
```
---

###  **Firebase AuthState Sınıfı**

Kullanıcı işlemleri sırasında farklı durumları kontrol etmek için bir `AuthState` sınıfı tanımlıyoruz. Bu sınıf, işlem sırasında başarılı, hata veya yükleniyor durumlarını yönetir.

```
sealed class AuthState {
    object Idle : AuthState()
    object Loading : AuthState()
    data class Success(val message: String) : AuthState()
    data class Error(val message: String?) : AuthState()
}
```

## Kullanıcı Kayıt ve Giriş Ekranı

Kullanıcı kaydını ve giriş işlemlerini yapmak için örnek bir ekran yapısı kullanabilirsiniz. Kullanıcıdan e-posta ve şifre alarak Firebase Authentication API'sini çağırıyoruz.

### Kayıt Sayfası Kod Örneği

```kotlin
@Composable
fun RegisterPage(authViewModel: AuthViewModel, navController: NavController) {
    var email by remember { mutableStateOf("") }
    var password by remember { mutableStateOf("") }
    val authState by authViewModel.authState.observeAsState()

    Column(
        modifier = Modifier
            .fillMaxSize()
            .padding(32.dp)
    ) {
        Text("Kayıt Ol", fontSize = 36.sp)
        
        // E-posta TextField
        OutlinedTextField(
            value = email,
            onValueChange = { email = it },
            label = { Text("E-Posta") },
            modifier = Modifier.fillMaxWidth()
        )
        
        // Şifre TextField
        OutlinedTextField(
            value = password,
            onValueChange = { password = it },
            label = { Text("Şifre") },
            modifier = Modifier.fillMaxWidth(),
            visualTransformation = PasswordVisualTransformation()
        )
        
        // Kayıt Ol Butonu
        Button(
            onClick = { authViewModel.registerUser(email, password) },
            enabled = email.isNotEmpty() && password.isNotEmpty()
        ) {
            Text("Kayıt Ol")
        }
        
        // Durum Kontrolü (Yükleniyor, Başarılı, Hata)
        when (authState) {
            is AuthState.Loading -> CircularProgressIndicator()
            is AuthState.Success -> {
                LaunchedEffect(Unit) {
                    navController.navigate("home")
                }
            }
            is AuthState.Error -> {
                Text("Hata: ${(authState as AuthState.Error).message}")
            }
            else -> {}
        }
    }
}
```

## Firebase Authentication'a Bağlantı Testi

Firebase Authentication işlevselliğinin doğru çalıştığını test etmek için kayıt ve giriş işlemlerini gerçekleştirebilirsiniz. Firebase Console üzerinden kullanıcılarınızın durumu ve işlem sonuçlarını izleyebilirsiniz.
