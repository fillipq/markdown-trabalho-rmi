A ideia geral é esta:

              MÁQUINA 1                         MÁQUINA 2
             CLIENTE                           SERVIDOR
        ┌─────────────────┐               ┌──────────────────┐
        │ Interface gráfica│               │   Servidor RMI  │
        │      (GUI)       │     RMI       │                  │
        │                  │ ────────────► │ Regras do sistema│
        │ Funcionários     │ ◄──────────── │                  │
        │ Cargos           │               │       ↓          │
        │ Departamentos    │               │      H2          │
        │ Pagamentos       │               │    Banco de      │
        └─────────────────┘               │     dados        │
                                          └──────────────────┘

1. O que vocês precisam desenvolver
O tema é Folha de Pagamento.

O professor sugeriu quatro entidades:

Funcionário

Cargo

Departamento

Pagamento

Eu faria exatamente essas quatro, porque elas permitem demonstrar bem o funcionamento do sistema.

Por exemplo:

Funcionário
Funcionário
------------------
id
nome
cpf
dataNascimento
salario
cargo
departamento

Cargo
Cargo
------------------
id
nome
descricao
salarioBase

Departamento
Departamento
------------------
id
nome
descricao

Pagamento
Pagamento
------------------
id
funcionario
mes
ano
salarioBruto
descontos
salarioLiquido
dataPagamento

2. O banco de dados fica no servidor
Essa parte é muito importante.

O enunciado diz:

"banco de dados do lado do servidor"

Então vocês não devem fazer o cliente acessar diretamente o banco.

O correto é:

Cliente
   │
   │ chamada RMI
   ▼
Servidor RMI
   │
   │ JDBC
   ▼
Banco H2

Por exemplo, o usuário clica:

"Cadastrar funcionário"

No cliente.

O cliente chama remotamente:

servidor.cadastrarFuncionario(funcionario);

O servidor recebe isso e faz:

INSERT INTO funcionario ...

no H2.

Depois retorna uma resposta para o cliente.

3. Onde entra o RMI?
Essa é provavelmente a parte mais importante da avaliação.

Vocês precisam ter uma interface remota.

Por exemplo:

public interface FuncionarioService extends Remote {

    void cadastrarFuncionario(Funcionario funcionario)
        throws RemoteException;

    Funcionario buscarFuncionario(int id)
        throws RemoteException;

    List<Funcionario> listarFuncionarios()
        throws RemoteException;

    void excluirFuncionario(int id)
        throws RemoteException;
}

Essa interface será compartilhada entre cliente e servidor.

No servidor vocês implementam:

public class FuncionarioServiceImpl
        extends UnicastRemoteObject
        implements FuncionarioService {

    @Override
    public void cadastrarFuncionario(Funcionario funcionario)
            throws RemoteException {

        // salva no banco
    }
}

E registram o serviço no RMI.

4. O cliente precisa ter interface gráfica
O enunciado fala:

"interface gráfica do lado do cliente"

Então não basta fazer um programa de terminal.

Vocês precisam de alguma GUI.

Pode ser:

Java Swing

JavaFX

outra tecnologia, caso o professor aceite.

Se vocês já estão usando Java, Swing provavelmente será o caminho mais simples.

Por exemplo, uma tela inicial:

┌─────────────────────────────────────┐
│       SISTEMA DE FOLHA              │
├─────────────────────────────────────┤
│                                     │
│  [ Funcionários ]                   │
│  [ Cargos ]                         │
│  [ Departamentos ]                  │
│  [ Pagamentos ]                     │
│                                     │
└─────────────────────────────────────┘

5. O que cada tela pode fazer
Vocês não precisam criar um sistema gigantesco. O importante é ter funcionalidades suficientes para demonstrar o RMI funcionando.

Eu faria:

Funcionários
Tela:

┌──────────────────────────────────────┐
│ Funcionários                          │
├──────────────────────────────────────┤
│ Nome:          [________________]     │
│ CPF:           [________________]     │
│ Salário:       [________________]     │
│ Cargo:         [______________▼]     │
│ Departamento:  [______________▼]     │
│                                      │
│ [Cadastrar] [Editar] [Excluir]       │
│                                      │
│ ┌──────────────────────────────────┐ │
│ │ ID │ Nome │ Cargo │ Departamento │ │
│ │ 1  │ João │ Dev   │ TI           │ │
│ │ 2  │ Maria│ RH    │ RH           │ │
│ └──────────────────────────────────┘ │
└──────────────────────────────────────┘

As operações podem ser:

cadastrar

listar

buscar

editar

excluir

6. Cargos
Uma tela semelhante:

Cargo
-----------------------
Nome
Descrição
Salário base

[Cadastrar]
[Editar]
[Excluir]
[Listar]

7. Departamentos
Por exemplo:

Departamento
-----------------------
Nome
Descrição

[Cadastrar]
[Editar]
[Excluir]
[Listar]

8. Pagamentos
Essa parte é especialmente interessante porque demonstra a regra de negócio.

Por exemplo:

Funcionário: João
Salário bruto: R$ 4.000,00

Descontos:
INSS: R$ 440,00
Outros: R$ 100,00

Salário líquido:
R$ 3.460,00

O cliente solicita:

pagamentoService.calcularPagamento(funcionarioId, mes, ano);

O servidor realiza o cálculo e salva o pagamento no banco.

Depois o cliente recebe o resultado.

9. Estrutura que eu recomendo
Como vocês precisam rodar em duas máquinas, eu separaria o projeto assim:

FolhaPagamento/
│
├── common/
│   ├── Funcionario.java
│   ├── Cargo.java
│   ├── Departamento.java
│   ├── Pagamento.java
│   │
│   ├── FuncionarioService.java
│   ├── CargoService.java
│   ├── DepartamentoService.java
│   └── PagamentoService.java
│
├── servidor/
│   ├── ServidorRMI.java
│   │
│   ├── FuncionarioServiceImpl.java
│   ├── CargoServiceImpl.java
│   ├── DepartamentoServiceImpl.java
│   ├── PagamentoServiceImpl.java
│   │
│   ├── Database.java
│   └── ...
│
├── cliente/
│   ├── ClienteRMI.java
│   │
│   ├── TelaPrincipal.java
│   ├── TelaFuncionario.java
│   ├── TelaCargo.java
│   ├── TelaDepartamento.java
│   └── TelaPagamento.java
│
└── banco/
    └── script.sql

Não precisa ser exatamente assim, mas separar cliente, servidor e código compartilhado facilita bastante.

10. O H2 que conversamos antes entra aqui
No computador do servidor:

Servidor
   │
   ├── ServidorRMI
   │
   ├── Classes de serviço
   │
   └── H2
        └── folha_pagamento.mv.db

O cliente não precisa instalar H2.

Inclusive, isso resolve exatamente o problema que você mencionou anteriormente sobre o computador da faculdade não ter MySQL.

Vocês podem usar H2 como banco, desde que o professor aceite a substituição do MySQL. Como o enunciado não especifica MySQL, apenas diz "banco de dados", H2 é uma escolha razoável.

11. Vocês também precisam entregar o script SQL
O professor explicitamente pediu:

"Script do Banco de Dados correspondente ao sistema."

Então não basta deixar o H2 criar as tabelas automaticamente.

Vocês devem entregar um arquivo:

script.sql

Por exemplo:

CREATE TABLE departamento (
    id INT AUTO_INCREMENT PRIMARY KEY,
    nome VARCHAR(100) NOT NULL,
    descricao VARCHAR(255)
);

CREATE TABLE cargo (
    id INT AUTO_INCREMENT PRIMARY KEY,
    nome VARCHAR(100) NOT NULL,
    descricao VARCHAR(255),
    salario_base DECIMAL(10,2)
);

CREATE TABLE funcionario (
    id INT AUTO_INCREMENT PRIMARY KEY,
    nome VARCHAR(150) NOT NULL,
    cpf VARCHAR(14) NOT NULL,
    salario DECIMAL(10,2),
    cargo_id INT,
    departamento_id INT,

    FOREIGN KEY (cargo_id)
        REFERENCES cargo(id),

    FOREIGN KEY (departamento_id)
        REFERENCES departamento(id)
);

CREATE TABLE pagamento (
    id INT AUTO_INCREMENT PRIMARY KEY,
    funcionario_id INT NOT NULL,
    mes INT NOT NULL,
    ano INT NOT NULL,
    salario_bruto DECIMAL(10,2),
    descontos DECIMAL(10,2),
    salario_liquido DECIMAL(10,2),
    data_pagamento DATE,

    FOREIGN KEY (funcionario_id)
        REFERENCES funcionario(id)
);

Depois vocês podem colocar alguns dados iniciais com INSERT.

12. A apresentação em duas máquinas é obrigatória
Essa parte é fundamental:

"Apresentação prática em duas máquinas (cliente e servidor)"

Então vocês precisam conseguir fazer algo assim:

Computador A
Servidor
IP: 192.168.0.10

ServidorRMI iniciado
Banco H2 iniciado
RMI aguardando conexões...

Computador B
Cliente
        ↓
conecta em 192.168.0.10
        ↓
Servidor RMI

E aí vocês demonstram.

Por exemplo:

No computador B:

Cadastrar funcionário

Nome: João
CPF: 123...
Cargo: Desenvolvedor
Departamento: TI
Salário: 4000

Clica em Cadastrar.

A chamada vai pela rede:

CLIENTE
   │
   │ cadastrarFuncionario()
   │
   ▼
REDE
   │
   ▼
SERVIDOR RMI
   │
   │ INSERT
   ▼
H2

Depois vocês podem voltar para o cliente e clicar em Listar funcionários.

O servidor consulta:

SELECT * FROM funcionario;

e devolve os dados através do RMI.

Isso é exatamente o tipo de coisa que o professor provavelmente quer ver na apresentação.

13. O que são os comentários sobre IAG?
Essa parte do enunciado é nova e importante.

Ele está dizendo que quando vocês utilizarem Inteligência Artificial Generativa, devem documentar isso nos comentários do código.

Por exemplo, antes de um método:

/*
 * IAG utilizada: ChatGPT
 *
 * Prompt:
 * "Crie um método Java para cadastrar um funcionário
 * utilizando JDBC e PreparedStatement."
 *
 * Validação humana:
 * O código foi analisado e testado pelo grupo.
 * Foram verificadas a conexão com o banco,
 * os parâmetros do PreparedStatement e o tratamento
 * das exceções.
 */
public void cadastrarFuncionario(Funcionario funcionario) {
    // ...
}

Vocês devem fazer isso conforme a orientação do professor/portaria mencionada no enunciado.

Não significa que vocês precisam colocar um comentário desses em absolutamente toda linha. A exigência citada fala, por exemplo, em comentário antes do método utilizado/gerado com IAG.

14. O que eu consideraria o "mínimo" para tirar uma boa nota
Eu montaria o projeto com:

Banco
 Tabela funcionario

 Tabela cargo

 Tabela departamento

 Tabela pagamento

 Chaves primárias

 Chaves estrangeiras

 Script SQL

Servidor
 RMI Registry

 Interfaces Remote

 Implementações dos serviços

 Conexão JDBC com H2

 CRUD de funcionário

 CRUD de cargo

 CRUD de departamento

 Cadastro/consulta de pagamento

 Regras de cálculo da folha

Cliente
 Interface gráfica

 Tela principal

 Tela de funcionários

 Tela de cargos

 Tela de departamentos

 Tela de pagamentos

 Cliente RMI

Apresentação
 Dois computadores

 Servidor em um computador

 Cliente no outro

 Comunicação pela rede

 Demonstrar uma operação passando pelo RMI

 Demonstrar que os dados são realmente persistidos no servidor

E eu faria uma coisa importante antes de vocês começarem
Não começaria pelas telas.

A ordem que eu recomendo é:

1. Banco de dados
       ↓
2. Classes das entidades
       ↓
3. Interfaces RMI
       ↓
4. Implementação do servidor
       ↓
5. Testar RMI localmente
       ↓
6. Cliente RMI
       ↓
7. Interface gráfica
       ↓
8. Testar em duas máquinas
       ↓
9. Organizar comentários de IAG
       ↓
10. Script SQL + apresentação

Se vocês quiserem, posso montar com vocês a estrutura completa desse trabalho, começando pelo banco e pelas classes Funcionario, Cargo, Departamento e Pagamento, e depois fazemos servidor RMI + H2 + cliente GUI passo a passo.



