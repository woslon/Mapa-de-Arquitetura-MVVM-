MVVM (Model - View - ViewModel)

Regras:

UI não chama API diretamente

ViewModel controla estado

Repository busca dados

API faz requisição HTTP

Model define estrutura


###### Estrutura de Pastas  ######

app/
 ├── model/
 │    └── User.kt
 │
 ├── data/
 │    ├── api/
 │    │     ├── ApiService.kt
 │    │     └── RetrofitInstance.kt
 │    │
 │    └── repository/
 │          └── UserRepository.kt
 │
 ├── viewmodel/
 │    └── MainViewModel.kt
 │
 ├── ui/
 │    ├── TelaA.kt
 │    └── TelaB.kt
 │
 └── App.kt


######### Diagrama da Arquitetura #########


        ┌────────────────────┐
        │   Jetpack Compose  │
        │        UI          │
        └─────────┬──────────┘
                  │
                  ↓
        ┌────────────────────┐
        │    ViewModel       │
        │  (Estado + lógica) │
        └─────────┬──────────┘
                  │
                  ↓
        ┌────────────────────┐
        │    Repository      │
        │ (Regra de dados)   │
        └─────────┬──────────┘
                  │
                  ↓
        ┌────────────────────┐
        │   Retrofit API     │
        │ (HTTP Requests)    │
        └─────────┬──────────┘
                  │
                  ↓
        ┌────────────────────┐
        │      Model         │
        │   (User, etc)      │
        └────────────────────┘


Android MVVM (Compose)

------------------------------------------------------------------------

model/User.kt

 # definir a tipagem

data class User(
  val id: Int,
  val name: String 
)

------------------------------------------------------------------------

data/api/ApiService.kt

# definir as rotas 

import retrofit2.http.GET

interface ApiService {

    @GET("user") # endpoint HTTP GET /user
    suspend fun getUser(): User # retorna um objeto User

}

------------------------------------------------------------------------

data/api/RetrofitInstance.kt

import retrofit2.Retrofit 
import retrofit2.converter.gson.GsonConverterFactory

object RetrofitInstance {

    val api: ApiService by lazy {
        Retrofit.Builder()
            .baseUrl("https://seuservidor.com/") # URL base da API
            .addConverterFactory(GsonConverterFactory.create()) # converte JSON → objeto
            .build()
            .create(ApiService::class.java) # cria implementação da API
    }

}

------------------------------------------------------------------------

data/repository/UserRepository.kt

class UserRepository {

    suspend fun getUser(): User {
        return RetrofitInstance.api.getUser() # chama a API
    }

}

------------------------------------------------------------------------

viewmodel/MainViewModel.kt

class MainViewModel : ViewModel() {

    private val repository = UserRepository() # acesso aos dados

    val user = mutableStateOf<User?>(null) # estado da UI

    fun carregar() {
        viewModelScope.launch { # executa em background
            user.value = repository.getUser() # atualiza estado
        }
    }

}

------------------------------------------------------------------------

AppNavigation.kt

@Composable fun App() {

    val nav = rememberNavController() # controlador de navegação
    val vm: MainViewModel = viewModel() # ViewModel compartilhado

    NavHost(nav, startDestination = "a") {
        composable("a") { TelaA(vm, nav) } # rota A
        composable("b") { TelaB(vm) }      # rota B
    }

}

------------------------------------------------------------------------

ui/TelaA.kt

@Composable fun TelaA( vm: MainViewModel, nav: NavController ) {

    Button(onClick = {
        vm.carregar() # chama ViewModel
        nav.navigate("b") # navega para outra tela
    }) {
        Text("Carregar")
    }

}

------------------------------------------------------------------------

ui/TelaB.kt

@Composable fun TelaB( vm: MainViewModel ) {

    val user = vm.user.value # observa estado

    Text("Nome: ${user?.name}") # mostra na tela

}

------------------------------------------------------------------------

