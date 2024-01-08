# Web APIs



Uma API Web é um conjunto de funções e procedimentos que permitem a criação de aplicativos que acessam os recursos ou dados de um sistema operacional, aplicativo ou outro serviço.



## Arquitetura do Sistema



- **Backend**:
  - Serviço responsável  por garantir a integridade dos dados, nomeadamente que todas as regras de domínio são respeitadas - lógica de negócio;
  - Server-side
- **Frontend**:
  - Responsável pela interação com o utilizador
  - Client-side
- A interação entre *backend* e *frontend* é feita através de uma HTTP API, disponibilizada pelo serviço de *backend*
- A iniciativa de comunicação está sempre do lado do cliente da API;
- O serviço de *backend* apenas comunica informações ao serviço de *frontend* no contexto de uma resposta HTTP;
- Implica *polling* pela aplicação *frontend* para verificar alterações de estados assíncronos.

## Desenho de APIs



Handler (presentation layer) <---> Service (Business Logic Layer) <---> Repository (Data Access Layer) <----> RDBMS

- Handler
  - 



