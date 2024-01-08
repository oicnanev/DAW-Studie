# DAW

## Contexto Spring

> é um contentor que instância, configura e gere um conjunto de objetos (beans) defenidos através de metadados de configuração

- *Injeção de Dependências (Dependecy Injection)* -  criação de um grafo de instâncias de componentes através de um conjunto de informações de tipos e suas dependências
- Sistema  que permite gerir dinâmicamente a composição e o ciclo de vida tanto dos objectos como das suas dependẽncias
- O *context* é o componente *Spring* por instânciar e gerir instâncias como as instâncias *controller*

### Injeção de Dependências (DI) e Inversão de Controlo (IoC)

- criação do grafo de objetos da aplicação
- Instâncialização dos objetos e suas dependências, injetando (passando como argumento) as dependências nos objetos
- O objeto dependente recebe e usa as dependências mas não a cria
- **Contructor injection** - injeção de dependências através do construtor (a forma preferível de injeção de dependências)
- **Field/Property injection** - as dependências são passadas após a criação do objeto, por meio da atribuição de campos ou da execução de métodos *setter* de propriedade:
  - requer campos de dependência mutáveis
  - os objeto não estão protos para operar após a construção
  - usar apenas quando a injeção no construtor não for possível
- **Wiring** - establece a dependência entre objetos
- **Class stereotype** - contentores de funcionalidades (componentes) vs contentores de dados

------

### Exemplos

Passos:

1. Criação do contexto
2. adicionar a anotação ```@Bean``` ao contexto
3. *refresh* do contexto, para ter em consideração as novas defenições de *bean*
4. usar o contexto para obter os *beans*

**bean** - objeto criado e gerido pelo *context*

#### 1º exemplo

Considerando as seguintes classes e dependências (dependente -> dependência):

- ComponentC -> ComponentB -> ComponentA

```kotlin
 // Create the context
val context = AnnotationConfigApplicationContext()

// Add the bean definitions
context.register(
    ComponentA::class.java,
    ComponentB::class.java,
    ComponentC::class.java
)

// Refresh the context
context.refresh()

// Get the beans (example: ComponentB)
val componentB = context.getBean<ComponentB>()
```

- Se o ComponentA não fosse registrado, o contexto não seria capaz de criar o ComponentB, lançando uma exceção;
- Se o ComponentB não fosse registrado, o contexto não seria capaz de criá-lo, lançando uma exceção.

#### 2º exemplo

É possível adicionar as definições de ```bean``` dadas a um pacote, usando o método ```scan```. Neste caso, o contexto irá procurar as classes anotadas com ```@Component``` e registrá-las:

```kotlin
// Create the context
val context = AnnotationConfigApplicationContext()

// Add the bean definitions
context.scan("package.name")
```

**Nota** também é possível usar o contexto com componentes inicializados com listas de dependências. Isto pode ser útil, por exemplo ao criar um *router*, onde se pode ter uma lista de controladores e injetá-los no *router*

#### 3º exemplo

Se um componente registado no contexto não for inicializado por um construtor, mas por um método estático, é possível usar a anotação ```@Bean``` para registá-lo

Por exemplo, um ```HttpClient``` é inicializado com o método ```newBuilder```, então criamos uma função que retorna um ```HttpClient``` e o anotamos com ```@Bean```:

```kotlin
@Configuration
class BeanConfig {
    @Bean
    fun httpClient(cookieHandler: CookieHandler): HttpClient = HttpClient
        .newBuilder()
        .cookieHandler(cookieHandler)
        .build()
}

// Create the context
val context = AnnotationConfigApplicationContext()

// Add the bean definitions
context.register(BeanConfig::class.java)
```

### Configuração de contentores baseada em anotações

- A anotação ```@Configuration``` indica que uma classe declara um ou mais métodos ```@Bean``` e pode ser processada pelo contentor **Spring** para gerar definições de **bean** e solicitações de serviço para esses *beans* em tempo de execução;
- A anotação ```@Bean``` informa o **Spring** que um método anotado com ```@Bean``` retornará um objeto que deve ser registrado como um **bean** no contexto da aplicação Spring;
- A anotação ```@Autowired``` é usada para *autowire bean* no método *setter*, construtor ou uma propriedade;
- A anotação ```@Component``` é um estereótipo genérico para qualquer componente gerenciado pelo **Spring**;
  - A anotação ```@Service``` é uma especialização da anotação ```@Component``` para a camada de serviço;
  - A anotação ```@Repository``` é uma especialização da ```@Component``` anotação para a camada de persistência;
  - A anotação ```@Controller``` é uma especialização da ```@Component``` anotação para a camada de apresentação (verifique a próxima seção para obter mais detalhes).

## Spring MVC

### Servlet

Um servlet é uma classe da linguagem  Java que é usada para estender os recursos de ```servers``` que hospedam aplicações acessíveis através dum modelo de ```request-response```.

- Permitir que manipuladores de solicitação (servlets) e intermediários (filtros) sejam executados em vários servidores web, os chamados contentores de servlet;
- Manipulação final para uma solicitação HTTP;
  - ```HttpServletRequest``` - representa as informações da solicitação;
  - ```HttpServletResponse``` - representa as informações de resposta produzidas até ao momento;
  - Ambos ```HttpServletRequest``` e ```HttpServletResponse```são únicos para cada par ```request-response```

### Filter

- Intermediário que para a realização de:
  - **Pré-processamento**, antes que o par ```request-response``` seja passado para o próximo intermediário ou manipulador final;
  - **Pós-processamento**, após o próximo intermediário ou manipulador final retornar
  - **Curto-circuito** (short-circuit), no processamento da solicitação, não chamando o próximo intermediário ou manipulador final.

### Design

- Uso de um servidor servlet subjacente, como Jetty, Tomcat, etc;
- Um servlet de despacho (dispatch servlet) registrado sobre esse servidor de servlet, para todos os caminhos abaixo de um caminho base;
- Manipuladores de ```requests``` que são responsáveis pelo processamento de solicitações para mapeamentos de solicitação específicos - uma maneira de definir manipuladores é por meio de métodos de instância de uma classe de controlador;

#### Argument Binding

- Mecanismo pelo qual o Spring MVC calcula os argumentos a serem passados para os manipuladores;
- Um ```argument resolver``` é um componente que é responsável por resolver um parâmetro de método num valor de argumento de uma determinada solicitação;
- Um controlador é uma classe anotada com ```@Controller``` ou ```@RestController``` que contém métodos de tratamento de solicitações;
  - Um controlador é **singleton** por padrão, isso significa que a mesma instância é usada para todas as solicitações; 
    - Isso acontece porque o Spring MVC segue o padrão **Singleton**, que garante que apenas uma instância de um objeto exista por aplicação. 
  - **Nota**: A anotação ```@RestController``` é uma especialização da anotação ```@Controller```, com o comportamento adicionado de que o valor de retorno de cada método é serializado automaticamente em JSON e passado de volta para o objeto ```HttpServletResponse```.
- Um ```handler``` é um método de instância dentro de uma classe de controlador, anotado com ```@RequestMapping``` ou ```@GetMapping```, ```@PostMapping```, etc;
  - O mesmo ```handler``` pode ser chamado simultaneamente por vários *threads*, uma vez que cada solicitação é tratada por um *thread* diferente;
- **Spring MVC** faz uso da Reflecção Java para localizar:
  - todas as classes que são controladores, através da presença da anotação ```@RestController``` ou ```@Controller```;
  - todos os métodos que são ```handlers```, através da presença da anotação ```@<Method>Mapping```.

#### Conversão de Mensagens

- Mecanismo responsável pela tradução dos ```payloads``` das mensagens HTTP em objetos e vice-versa;
- O Spring MVC fornece suporte integrado para o formato JSON, usando a biblioteca Jackson;
- É possível adicionar suporte para outros formatos, usando a interface ```HttpMessageConverter```.

#### Pipeline Processing

Há duas maneiras distintas de adicionar código a ser executado antes e depois da execução de um ```handler```:

**Servlet Filters**

- Mecanismo definido pela API Java Servlet;
- Permite que código seja executado antes e depois de a solicitação ser processada pelo servlet;
- Também podem curto-circuitar (short-circuit) o processamento de um ```request``` , encerrando o ciclo ```request-response```;
- Não têm acesso a informações específicas do Spring, como o ```handler``` que irá ser executado.

**Handler Interceptors**

- Mecanismo específico do Spring para intercetar a chamada para o ```handler``` de solicitação, ou seja, para ter o código executado antes e depois que o ```handler``` ser executado;
- Podem aceder a informações específicas do ```handler```;
- **Desvantagem**: São executados mais tarde no pipeline, após o ```dispatch servlet```.

#### manuseamento de Excepções

- A anotação ```@ControllerAdvice``` permite definir uma classe que será usada para lidar com exceções lançadas por ```handlers```;
- Existem duas formas de representar erros numa API:
  - **Baseado em excepções**:
    -  O ```handler``` lança uma exceção;
    - A exceção é tratada por uma anotação de classe ```@ControllerAdvice```
    - A exceção é traduzida numa resposta HTTP;
  - **Baseada em retorno**:
    - O ```handler```retorna um objecto ```ResponseEntity``` 
    - O objeto ```ResponseEntity``` é traduzido numa ```HTTP response```

### Perguntas e respostas de exame

#### Escolha múltipla

> Na platforma Spring MVC, por omissão, o construtor de uma classe anotada com @RestController é chamado:
  i. Uma vez por cada pedido HTTP, independentemente do handler que processa o pedido. 
  ii. Uma vez por cada pedido HTTP processado por um handler presente nessa classe. 
  iii. Uma vez por cada utilizador distinto. 
  iv. Uma vez por cada instância da aplicação.

Resposta iv

O construtor de uma classe anotada com @RestController na plataforma Spring MVC é chamado uma vez por cada pedido HTTP processado por um handler presente nessa classe

------

> Na platforma Spring MVC e assumindo a configuração por omissão:
  i. Os handlers presentes numa classe controller podem ser chamados em concorrência sobre a mesma instância, por threads distintas. 
  ii. Os handlers presentes numa classe controller nunca são chamados em concorrência sobre a mesma instância, porque apenas existe uma thread a processar pedidos. 
  iii. Os handlers presentes numa classe controller nunca são chamados em concorrência sobre a mesma instância, porque existe um lock que protege esses acessos.
  iv. Nenhuma das anteriores.

Resposta i

Os handlers presentes numa classe controller podem ser chamados em concorrência sobre a mesma instância, por threads distintas.

------

> Na plataforma Spring MVC, por omissão, uma classe anotada com @RestController vai ter as seguintes instâncias criadas pelo contexto Spring: 
 i. Uma única instância.
 ii. Uma instância distinta por pedido. 
 iii. Uma instância distinta por cada ligaçãao de um cliente. 
 iv. O menor número de instâncias, de

Resposta i

Por padrão, os controladores Spring MVC são singletons. Isso significa que uma única instância do controlador é criada e compartilhada entre todas as solicitações e sessões.

------

> No contexto da plataforma Spring MVC i. Vai existir uma instância implementando a interface HttpServletRequest partilhada por todos os pedidos. ii. Vai existir uma instância implementando a interface HttpServletRequest partilhada por todos os pedidos para o mesmo controlador. iii. Vai existir uma instância implementando a interface HttpServletRequest partilhada por todos os pedidos para o mesmo handler. iv. Nenhuma das anteriores.

Resposta iv

Explicação:

- No contexto da plataforma Spring MVC, o HttpServletRequest é um objeto que representa a requisição HTTP feita pelo cliente (navegador) ao servidor.
- Cada pedido HTTP cria uma nova instância de HttpServletRequest.
- Essa instância contém informações sobre o pedido, como parâmetros, cabeçalhos, método HTTP, URL, etc.
- Não há compartilhamento de instâncias entre diferentes pedidos. Cada pedido tem sua própria instância.

Em resumo, cada pedido HTTP cria uma nova instância de HttpServletRequest, e não há compartilhamento entre pedidos. 😊

------

> Na biblioteca Spring MVC e na configuração por omissão:
  i. E sempre criada uma nova instância dum interceptor por cada pedido. 
  ii. Não são criadas instâncias de interceptors porque todos os métodos têm de ser estáticos. 
  iii. Apenas são criadas novas instâncias de interceptors quando todas as instâncias existentes estiverem a ser usadas. 
  iv. Nenhuma das anteriores.
  
Resposta iii

Explicação:

- Na biblioteca Spring MVC, os interceptors são criados como beans geridos pelo contentor Spring.
- Por padrão, o Spring cria uma única instância de cada interceptor e a reutiliza-a para todos os pedidos.
- Essa abordagem é eficiente e evita a criação desnecessária de instâncias.

Em resumo, a opção iii é a correta porque o Spring reutiliza as instâncias de interceptors existentes, criando novas apenas quando necessário. 😊

------

> No contexto da utilização da biblioteca Spring MVC, a execução da funçãao doFilter pertencente à interface HttpFilter: 
 i. Ocorre sempre no contexto da mesma thread. 
 ii. Ocorre sempre no contexto da thread associada à instância sobre a qual é chamado o método. 
 iii. Ocorre sempre no contexto da thread associada ao pedido HTTP que resultou nesta chamada. 
 iv. Ocorre sempre no contexto da thread associada ao handler que vai processar o pedido HTTP.

Resposta iii

No contexto da utilização da biblioteca Spring MVC, a execução da função doFilter pertencente à interface HttpFilter ocorre sempre no contexto da thread associada ao pedido HTTP que resultou nesta chamada. O método doFilter é usado para interceptar solicitações HTTP e manipular a resposta antes de enviá-la de volta ao cliente. O método doFilter é chamado pelo contentor do servlet para permitir que um filtro processe a solicitação e a resposta. O contentor do servlet cria uma nova thread para cada solicitação HTTP recebida e chama o método doFilter nessa thread.

------

> Na biblioteca Spring MVC e na configuração por omissão: 
 i. E usada apenas uma instância por cada tipo de controller. 
 ii. E criada sempre uma nova instância dum controller por cada pedido. 
 iii. Apenas são criadas novas instâncias de controller quando todas as instâncias existentes estiverem a ser usadas. 
 iv. Não são criadas instâncias de controller porque todos os métodos têm de pertencer ao companion object.
 
Resposta i

Na biblioteca Spring MVC e na configuração por omissão, é usada apenas uma instância por cada tipo de controller. Isso significa que, por padrão, o contentor do Spring cria apenas uma instância de cada classe de controlador e a reutiliza para todos os pedidos HTTP recebidos pela aplicação. Essa abordagem é conhecida por escopo de singleton e é usada para garantir que os controladores sejam eficientes e escaláveis

------

#### Desenvolvimento

> No contexto da plataforma Spring MVC, indique duas formas distintas para a definição de beans.

**@Component**:

A anotação @Component é usada para varredura automática e configuração automática de beans. Ela permite que o Spring detecte e configure automaticamente os beans com base na varredura do classpath. Cada classe anotada com @Component (ou suas variantes, como @Service e @Repository) é mapeada implicitamente para um bean (ou seja, um bean por classe). Essa abordagem é conveniente e eficaz para a maioria dos casos, especialmente quando você possui o código-fonte das classes. Exemplo:

```kotlin
import org.springframework.stereotype.Component;

@Component
public class MeuBean {
    // Implementação do bean
}
```

**@Bean**:

A anotação @Bean é usada para declarar explicitamente um bean no Spring. Ela permite que você configure e crie beans de acordo com suas necessidades específicas. A anotação @Bean é aplicada a métodos, não a classes. Esses métodos retornam objetos que o Spring registra como beans no contexto da aplicação. Você pode usar @Bean em uma classe anotada com @Configuration ou @Component. Exemplo:

```kotlin
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class AppConfig {
    @Bean
    public MeuBean meuBean() {
        return new MeuBean();
    }
}
```

**Diferenças**

@Component é usado para componentes detectados automaticamente, enquanto @Bean é usado para declarar explicitamente um bean. 
@Component é mais simples e automático, enquanto @Bean oferece maior controle sobre a configuração do bean. 
Usa-se @Component quando se tem código-fonte da classe e se deseja que o Spring cuide da configuração. 
Usa-se @Bean quando é necessário configurar manualmente um bean ou quando se trabalha com bibliotecas de terceiros sem acesso ao código-fonte.

> No contexto da plataforma Spring MVC, quais os critérios que se deve ter em conta para escolher um servlet filter ou um handler interceptor como ponto de extensibilidade a usar para criar um intermediário no processamento de pedidos.

No contexto da plataforma **Spring MVC**, a escolha entre usar um **servlet filter** ou um **handler interceptor** como ponto de extensibilidade para criar um intermediário no processamento de pedidos depende de vários critérios. Vamos analisar esses critérios:

1. **Escopo e Propósito**:
   - **Servlet Filter**:
     - Os filtros são parte do servidor web (como Tomcat, Jetty etc.) e não fazem parte do framework Spring.
     - Eles operam em um nível mais baixo, antes que as solicitações alcancem qualquer componente do Spring MVC.
     - Filtros podem ser aplicados a todas as solicitações e respostas, independentemente do contexto do Spring.
     - São ideais para tarefas de nível mais geral, como autenticação, log e compressão de dados.
   - **Handler Interceptor**:
     - Os interceptores fazem parte do framework Spring MVC e operam no nível do DispatcherServlet.
     - São específicos para o processamento de solicitações relacionadas a controladores (handlers) do Spring.
     - Permitem a manipulação de solicitações antes que elas alcancem os controladores e também após a renderização da visão.
     - São adequados para tarefas específicas do Spring MVC, como logging, autorização, manipulação de modelos, etc.

2. **Ordem de Execução**:
   - **Servlet Filter**:
     - A ordem de execução dos filtros é definida no arquivo `web.xml` ou por meio de anotações.
     - A ordem é importante, pois os filtros são executados na sequência em que são declarados.
   - **Handler Interceptor**:
     - A ordem de execução dos interceptores é configurada no contexto do Spring MVC.
     - Os interceptores são executados na ordem em que são registrados.

3. **Acesso ao Contexto do Spring**:
   - **Servlet Filter**:
     - Os filtros não têm acesso direto ao contexto do Spring.
     - Se você precisar de recursos específicos do Spring, como beans gerenciados pelo contêiner, não poderá acessá-los diretamente nos filtros.
   - **Handler Interceptor**:
     - Os interceptores têm acesso ao contexto do Spring.
     - Eles podem injetar beans do Spring e usar recursos específicos do Spring.

4. **Granularidade e Flexibilidade**:
   - **Servlet Filter**:
     - São mais granulares e podem ser aplicados globalmente a todas as solicitações.
     - Oferecem maior flexibilidade, mas também maior complexidade.
   - **Handler Interceptor**:
     - São mais específicos para controladores e permitem manipulação mais focada.
     - São mais fáceis de configurar e usar.

Em resumo, escolha um **servlet filter** quando precisar de funcionalidades globais ou de nível mais baixo, e um **handler interceptor** quando precisar de manipulação específica do Spring MVC. Ambos têm seus casos de uso e devem ser escolhidos com base nos requisitos específicos do seu aplicativo.

------

> No âmbito da biblioteca Spring, indique qual a consequência de anotar classes com a anotação @Component.

A anotação @Component é uma forma de indicar ao Spring que uma classe é um bean gerenciado pelo contentor de inversão de controle (IoC) da framework. Isso significa que, sem ter que escrever código explícito, o Spring irá:

- Percorrer a aplicação em busca de classes anotadas com @Component
- Instanciar essas classes e injetar as dependências especificadas nelas
- Injetar essas classes onde forem necessárias

A anotação @Component é uma das chamadas anotações estereotipadas do Spring, que servem para marcar as classes como candidatas à detecção automática e configuração baseada em anotações. Outras anotações estereotipadas são @Repository, @Service e @Controller, que são especializações de @Component com usos e significados específicos fora do contexto de auto-detecção ou injeção de dependência

------

#### Código

> Realize um ou mais componentes para a plataforma Spring MVC de forma a expôr um recurso no caminho /handlers. Um pedido de método GET a este recurso deve retornar um objecto JSON. Cada campo deste objecto representa um handler usado no processamento de pelo menos um pedido, sendo o valor do campo um objecto com:
- O número de vezes que o handler foi utilizado. 
- O tempo médio de execução dos pedidos a esse handler.
Assuma que todos os handlers são do tipo HandlerMethod. 
Use o método getShortLogMessage para obter uma representação textual do handler. Valorizam-se soluções em que o cálculo do tempo de processamento inclua não só o tempo de execução do handler mas também o da maioria dos intermediários envolvidos (e.g. filtros e interceptores).

Para expor um recurso no caminho /handlers em uma aplicação Spring MVC, você pode criar um controlador com um método que retorna um objeto JSON com informações sobre os handlers usados no processamento de pelo menos um pedido. Cada campo do objeto representa um handler e contém o número de vezes que o handler foi utilizado e o tempo médio de execução dos pedidos a esse handler. Para obter uma representação textual do handler, você pode usar o método getShortLogMessage. Para calcular o tempo de processamento, você pode incluir o tempo de execução dos intermediários envolvidos, como filtros e interceptores.

```kotlin
@RestController
@RequestMapping("/handlers")
class HandlerInfoController @Autowired constructor(private val handlerMapping: RequestMappingHandlerMapping) {

    data class HandlerStats(var usageCount: Int = 0)

    @GetMapping
    fun getHandlerInfo(): Map<String, Map<String, Any>> {
        val result = mutableMapOf<String, Map<String, Any>>()
        val handlerStatsMap = mutableMapOf<HandlerMethod, HandlerStats>()

        // Itera sobre todos os handlers mapeados
        for ((_, handlerMethod) in handlerMapping.handlerMethods) {
            // Verifica se o handler já está no mapa de estatísticas
            if (!handlerStatsMap.containsKey(handlerMethod)) {
                handlerStatsMap[handlerMethod] = HandlerStats()
            }

            // Atualiza as estatísticas do handler
            val handlerStats = handlerStatsMap[handlerMethod]!!
            handlerStats.incrementUsageCount()
            // Aqui, você pode adicionar lógica para calcular o tempo de execução incluindo interceptors, etc.
        }

        // Converte as estatísticas para o formato desejado no JSON
        for ((handlerMethod, handlerStats) in handlerStatsMap) {
            val handlerInfo = mapOf(
                "usageCount" to handlerStats.usageCount
                // Adicione outros campos conforme necessário
            )

            result[handlerMethod.method.toGenericString()] = handlerInfo
        }

        return result
    }
}
```

------

> Realize um ou mais componentes para a plataforma Spring MVC de forma a expor um recurso no caminho /anonymous. Um pedido de método GET a este recurso deve retornar a lista contendo a contabilização de todos os acessos anónimos aos recursos da API. Cada elemento da lista é composto pelo URI do recurso e pelo número de acessos anónimos a esse recurso

Primeiro, um controlador que mapeie o caminho /anonymous e mantenha um registro dos acessos anônimos:

```kotlin
import org.springframework.stereotype.Controller
import org.springframework.web.bind.annotation.GetMapping
import org.springframework.web.bind.annotation.ResponseBody

@Controller
class AnonymousAccessController {

    // Mapa para registrar os acessos anônimos (URI -> Contagem)
    private val anonymousAccessMap = mutableMapOf<String, Int>()

    @GetMapping("/anonymous")
    @ResponseBody
    fun getAnonymousAccessStats(): Map<String, Int> {
        // Retorna o mapa de contagem de acessos anônimos
        return anonymousAccessMap
    }

    // Método para registrar um novo acesso anônimo
    fun registerAnonymousAccess(uri: String) {
        anonymousAccessMap[uri] = anonymousAccessMap.getOrDefault(uri, 0) + 1
    }
}
```

Agora, sempre que houver um acesso anónimo a um recurso, chamar o método registerAnonymousAccess para registrar o URI desse recurso:

```kotlin
@Controller
class MyController(private val anonymousAccessController: AnonymousAccessController) {
    // Quando ocorrer um acesso anônimo a um recurso, registre-o
	val uriAcessada = "/algum-recurso" // Substitua pelo URI real
	anonymousAccessController.registerAnonymousAccess(uriAcessada)
}
```

O endpoint /anonymous retornará o mapa com a contagem de acessos anônimos para cada URI. Por exemplo:

```json
{
    "/algum-recurso": 5,
    "/outro-recurso": 3,
    "/mais-um-recurso": 10
}
```

------

> Realize um ou mais componentes para a plataforma Spring MVC de forma a que todas as respostas produzidas tenham o header HTTP Debug contendo o tempo em milisegundos que a resposta demorou a ser processada, bem como o eventual handler envolvido nesse processamento. Valorizam-se soluções em que o cálculo do tempo de processamento inclua não só o tempo de execução do eventual handler mas também o da maioria dos intermediários envolvidos (e.g. filtros e interceptores)

Para adicionar um header HTTP de depuração contendo o tempo de processamento em milissegundos, incluindo o tempo de execução do handler e outros intermediários (como filtros e interceptores), podemos criar um interceptor personalizado no Spring MVC.

1. **Crie um Interceptor**:

   - Crie uma classe que implemente a interface `HandlerInterceptor`.
   - Sobrescreva os métodos `preHandle`, `postHandle` e `afterCompletion`.
   - Calcule o tempo de processamento no método `afterCompletion`.

2. **Exemplo de Interceptor**:
. 
   ```kotlin
   import org.springframework.web.servlet.HandlerInterceptor
   import org.springframework.web.servlet.ModelAndView
   import javax.servlet.http.HttpServletRequest
   import javax.servlet.http.HttpServletResponse
   
   class DebugHeaderInterceptor : HandlerInterceptor {
   
       override fun preHandle(request: HttpServletRequest, response: HttpServletResponse, handler: Any): Boolean {
           // Executa antes do handler (controlador)
           return true
       }
   
       override fun postHandle(request: HttpServletRequest, response: HttpServletResponse, handler: Any, modelAndView: ModelAndView?) {
           // Executa após o handler (controlador), mas antes da renderização da visão
       }
   
       override fun afterCompletion(request: HttpServletRequest, response: HttpServletResponse, handler: Any, ex: Exception?) {
           // Executa após a renderização da visão
           val startTime = request.getAttribute("startTime") as Long
           val endTime = System.currentTimeMillis()
           val processingTime = endTime - startTime
   
           response.addHeader("X-Debug-Processing-Time", "$processingTime ms")
           response.addHeader("X-Debug-Handler", handler.toString())
       }
   }```

3. **Registre o Interceptor**:

   - No seu arquivo de configuração (por exemplo, `WebMvcConfigurer`), registre o interceptor:
   - 
     ```kotlin
     import org.springframework.context.annotation.Configuration
     import org.springframework.web.servlet.config.annotation.InterceptorRegistry
     import org.springframework.web.servlet.config.annotation.WebMvcConfigurer
     
     @Configuration
     class WebConfig : WebMvcConfigurer {
     
         override fun addInterceptors(registry: InterceptorRegistry) {
             registry.addInterceptor(DebugHeaderInterceptor())
         }
     }
     ```

4. **Uso do Header**:
   
   - Agora, todas as respostas terão os headers `X-Debug-Processing-Time` e `X-Debug-Handler` contendo o tempo de processamento e o handler envolvido.

------

> Realize um ou mais componentes para a plataforma Spring MVC, de forma a que um recurso seja exposto no caminho /pending. Um pedido de método GET para esse recurso deve retornar uma mensagem com uma representação contendo um objeto JSON. Esse objeto deve representar o número de pedidos atualmente em processamento (i.e. iniciados e não concluídos), agrupados por método HTTP.

Componente Spring MVC em Kotlin para expor o recurso no caminho `/pending`. O objetivo é retornar um objeto JSON que represente o número de pedidos atualmente em processamento, agrupados por método HTTP.

1. **Crie um Controller**:
    - Vamos criar um controlador chamado `PendingController` que será responsável por lidar com as solicitações para o recurso `/pending`.

2. **Defina o Endpoint**:
    - O endpoint `/pending` responderá a solicitações de método GET.

3. **Contagem de Pedidos**:
    - Para este exemplo, vou simular uma contagem de pedidos em processamento. Você pode adaptar isso para sua lógica real.
    - Vou criar um objeto JSON com as contagens para cada método HTTP (GET, POST, PUT, DELETE).

Exemplo:

```kotlin
import org.springframework.web.bind.annotation.GetMapping
import org.springframework.web.bind.annotation.RequestMapping
import org.springframework.web.bind.annotation.RestController

@RestController
@RequestMapping("/pending")
class PendingController {

    // Simulando contagem de pedidos em processamento
    private val pendingRequests = mapOf(
        "GET" to 10,
        "POST" to 5,
        "PUT" to 3,
        "DELETE" to 2
    )

    @GetMapping
    fun getPendingRequests(): Map<String, Int> {
        return pendingRequests
    }
}
```

Neste exemplo:
- O endpoint `/pending` responde a solicitações de método GET.
- A contagem de pedidos em processamento é simulada no mapa `pendingRequests`.

Quando você acessar `http://localhost:8080/pending`, receberá uma resposta JSON como esta:

```json
{
    "GET": 10,
    "POST": 5,
    "PUT": 3,
    "DELETE": 2
}
```

------

> Realize um ou mais componentes para a plataforma Spring MVC, de forma a que um conjunto de recursos seja exposto no caminho /status/{method}. Um pedido de método GET para um desses recursos deve retornar uma mensagem com uma representaçã ao contendo um objeto JSON. Esse objeto deve representar o tempo máximo e mínimo do processamento de pedidos com método method. Caso o servidor ainda não tenha recebido pedidos com o método method, a resposta ao pedido deve ter status code 404

Componente Spring MVC em Kotlin para expor o conjunto de recursos no caminho `/status/{method}`. O objetivo é retornar um objeto JSON que represente o tempo máximo e mínimo de processamento de pedidos com o método especificado. Se o servidor ainda não tiver recebido pedidos com o método `method`, a resposta ao pedido deve ter o status code 404.

código de exemplo:

```kotlin
import org.springframework.http.HttpStatus
import org.springframework.http.ResponseEntity
import org.springframework.web.bind.annotation.GetMapping
import org.springframework.web.bind.annotation.PathVariable
import org.springframework.web.bind.annotation.RequestMapping
import org.springframework.web.bind.annotation.RestController

@RestController
@RequestMapping("/status")
class StatusController {

    // Simulando tempos de processamento (em milissegundos)
    private val processingTimes = mapOf(
        "GET" to listOf(100, 200, 300), // Exemplo: [100, 200, 300]
        // Adicione outros métodos e tempos de processamento aqui
    )

    @GetMapping("/{method}")
    fun getStatus(@PathVariable method: String): ResponseEntity<Map<String, Int>> {
        val times = processingTimes[method.toUpperCase()] ?: return ResponseEntity.notFound().build()
        val minTime = times.minOrNull() ?: 0
        val maxTime = times.maxOrNull() ?: 0
        val result = mapOf("minTime" to minTime, "maxTime" to maxTime)
        return ResponseEntity(result, HttpStatus.OK)
    }
}
```

Neste exemplo:
- O endpoint `/status/{method}` responde a solicitações de método GET com o método especificado.
- Os tempos de processamento são simulados no mapa `processingTimes`.
- Se o método não estiver presente no mapa, a resposta terá o status code 404 (Not Found).

Quando você acessar `http://localhost:8080/status/GET`, receberá uma resposta JSON como esta:

```json
{
    "minTime": 100,
    "maxTime": 300
}
```

------

> Realize um ou mais componentes para a plataforma Spring MVC, de forma a que um recurso seja exposto no caminho /failures. Um pedido de método GET para esse recurso deve retornar uma mensagem com uma representação contendo um objeto JSON. Esse objecto deve representar os últimos 10 pedidos que resultaram numa resposta com status code maior ou igual a 500. A representação de cada pedido deve incluir: o método do pedido, o URI do pedido, o status code da resposta, e o nome do controlador e do método responsável pelo processamento desse pedido (caso o processamento tenha sido realizado por um controlador).

podemos usar os seguintes componentes:

- Um **filtro** que intercepta todas as requisições e registra as informações relevantes em um **repositório**. O filtro também verifica se a resposta tem um status code maior ou igual a 500 e, nesse caso, adiciona o nome do controlador e do método responsável pelo processamento ao registro.
- Um **controlador** que expõe o recurso `/failures` e retorna um objeto JSON com os últimos 10 registros do repositório.

A seguir, apresentamos um possível código em Kotlin para esses componentes, usando a biblioteca **Spring Boot**¹ e o formato **JSON**²:

```kotlin
// Um data class que representa um registro de uma requisição e uma resposta
data class Failure(
    val method: String, // o método do pedido
    val uri: String, // o URI do pedido
    val status: Int, // o status code da resposta
    var controller: String? = null, // o nome do controlador responsável pelo processamento
    var handler: String? = null // o nome do método responsável pelo processamento
)

// Um repositório que armazena os registros em uma lista
@Component
class FailureRepository {
    private val failures = mutableListOf<Failure>()

    // Adiciona um novo registro à lista
    fun addFailure(failure: Failure) {
        failures.add(failure)
    }

    // Retorna os últimos 10 registros da lista
    fun getLastFailures(): List<Failure> {
        return failures.takeLast(10)
    }
}

// Um filtro que intercepta todas as requisições e registra as informações relevantes
@Component
class FailureFilter(
    @Autowired val failureRepository: FailureRepository // injeta o repositório
) : Filter {
    override fun doFilter(
        request: ServletRequest,
        response: ServletResponse,
        chain: FilterChain
    ) {
        // Converte os objetos de requisição e resposta para HttpServletRequest e HttpServletResponse
        val httpRequest = request as HttpServletRequest
        val httpResponse = response as HttpServletResponse

        // Cria um novo registro com as informações da requisição e da resposta
        val failure = Failure(
            method = httpRequest.method,
            uri = httpRequest.requestURI,
            status = httpResponse.status
        )

        // Adiciona o registro ao repositório
        failureRepository.addFailure(failure)

        // Verifica se a resposta tem um status code maior ou igual a 500
        if (httpResponse.status >= 500) {
            // Obtém o objeto HandlerExecutionChain que contém o controlador e o método responsável pelo processamento
            val handler = (request.getAttribute(HandlerMapping.HANDLER_ATTRIBUTE) as HandlerExecutionChain).handler

            // Verifica se o handler é um HandlerMethod
            if (handler is HandlerMethod) {
                // Adiciona o nome do controlador e do método ao registro
                failure.controller = handler.beanType.simpleName
                failure.handler = handler.method.name
            }
        }

        // Continua o processamento da requisição
        chain.doFilter(request, response)
    }
}

// Um controlador que expõe o recurso /failures
@RestController
class FailureController(
    @Autowired val failureRepository: FailureRepository // injeta o repositório
) {
    // Define um método GET para o caminho /failures
    @GetMapping("/failures")
    fun getFailures(): ResponseEntity<List<Failure>> {
        // Obtém os últimos 10 registros do repositório
        val failures = failureRepository.getLastFailures()

        // Retorna uma resposta com o objeto JSON dos registros
        return ResponseEntity.ok(failures)
    }
}
```

------

> Realize um ou mais componentes para uso com a biblioteca Spring MVC de forma a expor recursos nos caminhos /handlers e /handlers/{handler-id}. Um pedido de método GET a /handlers retorna uma representação JSON contendo uma lista de links para caminhos com a forma /handlers/{handler-id}, um para cada handler que tenha sido executado até ao momento. Um pedido para /handlers/{handler-id} retorna uma representação com o número de vezes que o handler foi executado, caso handler-id seja o identificador de um handler executado pelo menos uma vez. Assuma que todos os handlers são do tipo HandlerMethod e use o método getShortLogMessage para obter o identificador dum handler.

Para realizar essa tarefa, podemos usar os seguintes componentes:

- Um **interceptor** que intercepta todas as requisições e registra os handlers executados em um **repositório**. O interceptor também incrementa o número de vezes que cada handler foi executado.
- Um **controlador** que expõe os recursos `/handlers` e `/handlers/{handler-id}` e retorna uma representação JSON com a lista de links ou o número de execuções, respectivamente.

A seguir, apresentamos um possível código em Java para esses componentes, usando a biblioteca **Spring MVC**¹ e o formato **JSON**²:

```java
// Um data class que representa um registro de um handler e o número de execuções
public class HandlerRecord {
    private String id; // o identificador do handler
    private int count; // o número de execuções

    // Construtor, getters e setters omitidos
}

// Um repositório que armazena os registros em um mapa
@Component
public class HandlerRepository {
    private Map<String, HandlerRecord> records = new HashMap<>();

    // Adiciona ou atualiza um registro no mapa
    public void addOrUpdateRecord(HandlerMethod handler) {
        // Obtém o identificador do handler usando o método getShortLogMessage
        String id = handler.getShortLogMessage();
        // Verifica se o mapa já contém um registro com esse identificador
        if (records.containsKey(id)) {
            // Se sim, incrementa o número de execuções
            records.get(id).setCount(records.get(id).getCount() + 1);
        } else {
            // Se não, cria um novo registro com uma execução
            records.put(id, new HandlerRecord(id, 1));
        }
    }

    // Retorna a lista de registros do mapa
    public List<HandlerRecord> getRecords() {
        return new ArrayList<>(records.values());
    }

    // Retorna um registro específico do mapa, ou null se não existir
    public HandlerRecord getRecord(String id) {
        return records.get(id);
    }
}

// Um interceptor que intercepta todas as requisições e registra os handlers executados
@Component
public class HandlerInterceptor extends HandlerInterceptorAdapter {
    @Autowired
    private HandlerRepository repository; // injeta o repositório

    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        // Verifica se o handler é um HandlerMethod
        if (handler instanceof HandlerMethod) {
            // Adiciona ou atualiza o registro no repositório
            repository.addOrUpdateRecord((HandlerMethod) handler);
        }
    }
}

// Um controlador que expõe os recursos /handlers e /handlers/{handler-id}
@RestController
public class HandlerController {
    @Autowired
    private HandlerRepository repository; // injeta o repositório

    // Define um método GET para o caminho /handlers
    @GetMapping("/handlers")
    public ResponseEntity<List<String>> getHandlers() {
        // Obtém a lista de registros do repositório
        List<HandlerRecord> records = repository.getRecords();
        // Cria uma lista de links para os caminhos /handlers/{handler-id}
        List<String> links = new ArrayList<>();
        for (HandlerRecord record : records) {
            links.add("/handlers/" + record.getId());
        }
        // Retorna uma resposta com o objeto JSON da lista de links
        return ResponseEntity.ok(links);
    }

    // Define um método GET para o caminho /handlers/{handler-id}
    @GetMapping("/handlers/{handler-id}")
    public ResponseEntity<Integer> getHandler(@PathVariable("handler-id") String id) {
        // Obtém o registro específico do repositório
        HandlerRecord record = repository.getRecord(id);
        // Verifica se o registro existe
        if (record != null) {
            // Se sim, retorna uma resposta com o objeto JSON do número de execuções
            return ResponseEntity.ok(record.getCount());
        } else {
            // Se não, retorna uma resposta com status code 404 (Not Found)
            return ResponseEntity.notFound().build();
        }
    }
}
```

------

> Realize um ou mais componentes para uso com a biblioteca Spring MVC de forma a que, para cada pedido HTTP, seja emitida uma mensagem de log com: método HTTP; URI do recurso acedido; status code da resposta; tempo de processamento; identificador do handler que processou o pedido, caso o handler seja do tipo HandlerMethod. Valorizam-se soluções onde o cálculo do tempo de processamento tem em conta mais etapas desse processamento. Use o método getShortLogMessage para obter o identificador dum HandlerMethod

Para registrar uma mensagem de log com informações sobre cada pedido HTTP em Kotlin, podemos criar um **interceptor** personalizado que capture as informações relevantes e as registre em um arquivo de log. O interceptor pode ser implementado usando a interface **HandlerInterceptor** fornecida pela biblioteca Spring MVC ¹.

Aqui está um exemplo de como implementar um interceptor personalizado para registrar informações de log para cada pedido HTTP:

```kotlin
class LoggingInterceptor : HandlerInterceptor {

    companion object {
        private val logger = LoggerFactory.getLogger(LoggingInterceptor::class.java)
    }

    override fun preHandle(request: HttpServletRequest, response: HttpServletResponse, handler: Any): Boolean {
        request.setAttribute("startTime", System.currentTimeMillis())
        return true
    }

    override fun postHandle(request: HttpServletRequest, response: HttpServletResponse, handler: Any, modelAndView: ModelAndView?) {
        val startTime = request.getAttribute("startTime") as Long
        val endTime = System.currentTimeMillis()
        val processingTime = endTime - startTime
        val handlerId = getShortLogMessage(handler)
        val requestUri = request.requestURI
        val statusCode = response.status
        logger.info("HTTP Request: {} {} {} {}ms {}", request.method, requestUri, statusCode, processingTime, handlerId)
    }

    private fun getShortLogMessage(handler: Any): String {
        return if (handler is HandlerMethod) {
            "${handler.beanType.simpleName}.${handler.method.name}"
        } else {
            handler.toString()
        }
    }
}
```

Neste exemplo, o interceptor personalizado é implementado como uma classe Kotlin que implementa a interface **HandlerInterceptor** ¹. O método **preHandle** é chamado antes do processamento do pedido HTTP e é usado para registrar o tempo de início do processamento do pedido ¹. O método **postHandle** é chamado após o processamento do pedido HTTP e é usado para registrar as informações relevantes de log, incluindo o método HTTP, URI do recurso acessado, status code da resposta, tempo de processamento e identificador do handler que processou o pedido ¹. O método **getShortLogMessage** é usado para obter o identificador do handler que processou o pedido ¹.

Para usar o interceptor personalizado, podemos registrá-lo no arquivo de configuração do Spring MVC, como mostrado abaixo:

```kotlin
@Configuration
class AppConfig : WebMvcConfigurer {

    override fun addInterceptors(registry: InterceptorRegistry) {
        registry.addInterceptor(LoggingInterceptor())
    }
}
```

Neste exemplo, o interceptor personalizado é registrado no método **addInterceptors** da classe **AppConfig** ¹. Isso garante que o interceptor seja aplicado a todos os pedidos HTTP recebidos pelo aplicativo Spring MVC

------

> Realize um ou mais componentes para a plataforma Spring MVC, de forma a que um recurso seja exposto no caminho /errors. Um pedido de método GET para esse recurso deve retornar uma mensagem com uma representação contendo um objeto JSON. Esse objeto deve representar os pedidos processados nos últimos 10 minutos e que resultaram numa resposta com status code igual a 500 A representação de cada pedido deve incluir: o momento em que o pedido foi recebido, o método do pedido, o URI do pedido, e o nome do controlador e do método responsável pelo processamento desse pedido (caso o processamento tenha sido realizado por um controlador).

Exemplo de como implementar um componente Spring MVC em Kotlin que expõe um recurso no caminho `/errors` e retorna uma mensagem com uma representação contendo um objeto JSON. Esse objeto representa os pedidos processados nos últimos 10 minutos e que resultaram em uma resposta com status code igual a 500. A representação de cada pedido inclui o momento em que o pedido foi recebido, o método do pedido, o URI do pedido e o nome do controlador e do método responsável pelo processamento desse pedido (caso o processamento tenha sido realizado por um controlador):

```kotlin
import org.springframework.http.HttpStatus
import org.springframework.web.bind.annotation.GetMapping
import org.springframework.web.bind.annotation.RequestMapping
import org.springframework.web.bind.annotation.RestController
import java.time.LocalDateTime

@RestController
@RequestMapping("/errors")
class ErrorController {

    private val errors = mutableListOf<Error>()

    @GetMapping
    fun getErrors(): List<Error> {
        val tenMinutesAgo = LocalDateTime.now().minusMinutes(10)
        return errors.filter { it.timestamp.isAfter(tenMinutesAgo) }
    }

    fun addError(error: Error) {
        errors.add(error)
    }

    data class Error(
        val timestamp: LocalDateTime,
        val method: String,
        val uri: String,
        val controller: String?,
        val handler: String?
    )

    @RestControllerAdvice
    class ErrorControllerAdvice {

        @ExceptionHandler(HttpStatus.INTERNAL_SERVER_ERROR::class)
        fun handleInternalServerError(ex: Exception) {
            val error = Error(
                LocalDateTime.now(),
                "GET",
                "/errors",
                null,
                null
            )
            addError(error)
        }
    }
}
```

Nesta implementação, a classe `ErrorController` é um controlador Spring MVC que expõe um recurso no caminho `/errors` ¹. O método `getErrors` retorna uma lista de erros que foram processados nos últimos 10 minutos e que resultaram em uma resposta com status code igual a 500 ¹. O método `addError` é usado para adicionar um novo erro à lista de erros ¹. A classe `Error` é uma classe de dados que representa um erro e inclui o momento em que o pedido foi recebido, o método do pedido, o URI do pedido e o nome do controlador e do método responsável pelo processamento desse pedido (caso o processamento tenha sido realizado por um controlador) ¹. A classe `ErrorControllerAdvice` é um controlador de exceções que é usado para capturar exceções do tipo `HttpStatus.INTERNAL_SERVER_ERROR` e adicionar um novo erro à lista de erros ¹.

Para usar o componente `ErrorController`, basta injetá-lo em outros componentes Spring MVC e chamar o método `addError` sempre que ocorrer um erro com status code igual a 500.
