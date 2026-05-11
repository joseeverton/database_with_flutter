# CRUD de Clientes com SQLite usando MVC

## Sobre o projeto
Você pode usar este projeto como atividade prática da disciplina de Programação para Dispositivos Móveis.
A proposta será desenvolver um sistema simples de cadastro de clientes utilizando:
- Flutter
- SQLite
- Padrão MVC
- CRUD completo
- Organização em camadas

## Objetivo da Aula
Criar um aplicativo Flutter capaz de:
- Cadastrar clientes
- Listar clientes
- Editar clientes
- Excluir clientes

### Cada cliente possuirá:
- Nome
- Nome
- E-mail
- Telefone
- Profissão

## Estrutura do Projeto (MVC)
Explicar aos alunos que MVC separa responsabilidades.

- Camada ----------- Responsabilidade
- Model	------------- Representa os dados
- View	--------------- Interface visual
- Controller --------- Regras de negócio

## Estrutura de Pastas
```
lib/
│
├── controller/
│   └── cliente_controller.dart
│
├── database/
│   └── database_helper.dart
│
├── model/
│   └── cliente.dart
│
├── view/
│   ├── cliente_form_page.dart
│   └── cliente_list_page.dart
│
└── main.dart
```

## Como executar o projeto
Para executar o projeto, siga os passos abaixo:
1. Faça um clone desse repositório;
2. Entre na pasta do projeto;
3. Execute o comando `flutter run` na linha de comando.


##  Instalar Dependências
Para instalar as dependências do projeto, siga os passos abaixo:
1. Adicionar SQLite
```
flutter pub add sqflite
```
2. Adicionar path
```
flutter pub add path
```
### Explicando as Dependências
- sqflite:
Biblioteca responsável pelo SQLite no Flutter.
1. Ela permite:
- Criar banco
- Criar tabelas
- Inserir dados
- Atualizar dados
- Excluir dados

2. path
- Ajuda a montar o caminho correto do banco no dispositivo.

##  Criando o Model
```
lib/model/cliente.dart
```

```dart
class Cliente {
  int? id;
  String nome;
  String email;
  String telefone;
  String profissao;

  Cliente({
    this.id,
    required this.nome,
    required this.email,
    required this.telefone,
    required this.profissao,
  });

  Map<String, dynamic> toMap() {
    return {
      'id': id,
      'nome': nome,
      'email': email,
      'telefone': telefone,
      'profissao': profissao,
    };
  }

  factory Cliente.fromMap(Map<String, dynamic> map) {
    return Cliente(
      id: map['id'],
      nome: map['nome'],
      email: map['email'],
      telefone: map['telefone'],
      profissao: map['profissao'],
    );
  }
}
```

### Explicação do Model
- Classe Cliente
    - Representa os dados do cliente.
- toMap()
    - Transforma objeto em Map.
    - O SQLite trabalha com Map.
- fromMap()
    - Transforma dados do banco em objeto novamente.

## Criando o Banco de Dados
```
lib/database/database_helper.dart
```
```dart
import 'package:path/path.dart';
import 'package:sqflite/sqflite.dart';

class DatabaseHelper {
  static final DatabaseHelper instance = DatabaseHelper._init();

  static Database? _database;

  DatabaseHelper._init();

  Future<Database> get database async {
    if (_database != null) return _database!;

    _database = await _initDB('clientes.db');
    return _database!;
  }

  Future<Database> _initDB(String filePath) async {
    final dbPath = await getDatabasesPath();

    final path = join(dbPath, filePath);

    return await openDatabase(
      path,
      version: 1,
      onCreate: _createDB,
    );
  }

  Future _createDB(Database db, int version) async {
    await db.execute('''
      CREATE TABLE clientes (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        nome TEXT,
        email TEXT,
        telefone TEXT,
        profissao TEXT
      )
    ''');
  }
}
```
### Explicação do Banco
- Singleton
    - É um padrão de projeto que permite a criação de um único objeto de uma classe.
    

    ```dart
    static final DatabaseHelper instance
    ```

- openDatabase()
    - Abre ou cria o banco.
- CREATE TABLE
    - Cria tabela clientes.

## Criando o Controller
```
lib/controller/cliente_controller.dart
```
```dart
import 'package:cadastro_clientes/database/database_helper.dart';
import 'package:cadastro_clientes/model/cliente.dart';

class ClienteController {
  final dbHelper = DatabaseHelper.instance;

  Future<int> inserirCliente(Cliente cliente) async {
    final db = await dbHelper.database;

    return await db.insert(
      'clientes',
      cliente.toMap(),
    );
  }

  Future<List<Cliente>> listarClientes() async {
    final db = await dbHelper.database;

    final result = await db.query('clientes');

    return result.map((e) => Cliente.fromMap(e)).toList();
  }

  Future<int> atualizarCliente(Cliente cliente) async {
    final db = await dbHelper.database;

    return await db.update(
      'clientes',
      cliente.toMap(),
      where: 'id = ?',
      whereArgs: [cliente.id],
    );
  }

  Future<int> deletarCliente(int id) async {
    final db = await dbHelper.database;

    return await db.delete(
      'clientes',
      where: 'id = ?',
      whereArgs: [id],
    );
  }
}
```

### Explicação do Controller
O controller faz a comunicação entre:
- Interface
- Banco de dados

## Tela de Cadastro
```
lib/view/cliente_form_page.dart
```
```dart
import 'package:flutter/material.dart';
import '../controller/cliente_controller.dart';
import '../model/cliente.dart';

class ClienteFormPage extends StatefulWidget {
  final Cliente? cliente;

  const ClienteFormPage({super.key, this.cliente});

  @override
  State<ClienteFormPage> createState() => _ClienteFormPageState();
}

class _ClienteFormPageState extends State<ClienteFormPage> {
  final controller = ClienteController();

  final nomeController = TextEditingController();
  final emailController = TextEditingController();
  final telefoneController = TextEditingController();
  final profissaoController = TextEditingController();

  @override
  void initState() {
    super.initState();

    if (widget.cliente != null) {
      nomeController.text = widget.cliente!.nome;
      emailController.text = widget.cliente!.email;
      telefoneController.text = widget.cliente!.telefone;
      profissaoController.text = widget.cliente!.profissao;
    }
  }

  salvar() async {
    Cliente cliente = Cliente(
      id: widget.cliente?.id,
      nome: nomeController.text,
      email: emailController.text,
      telefone: telefoneController.text,
      profissao: profissaoController.text,
    );

    if (widget.cliente == null) {
      await controller.inserirCliente(cliente);
    } else {
      await controller.atualizarCliente(cliente);
    }

    Navigator.pop(context, true);
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text(
          widget.cliente == null
              ? 'Novo Cliente'
              : 'Editar Cliente',
        ),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            TextField(
              controller: nomeController,
              decoration: const InputDecoration(
                labelText: 'Nome',
              ),
            ),

            TextField(
              controller: emailController,
              decoration: const InputDecoration(
                labelText: 'E-mail',
              ),
            ),

            TextField(
              controller: telefoneController,
              decoration: const InputDecoration(
                labelText: 'Telefone',
              ),
            ),

            TextField(
              controller: profissaoController,
              decoration: const InputDecoration(
                labelText: 'Profissão',
              ),
            ),

            const SizedBox(height: 20),

            ElevatedButton(
              onPressed: salvar,
              child: const Text('Salvar'),
            ),
          ],
        ),
      ),
    );
  }
}
```
## Tela de Listagem
```
lib/view/cliente_list_page.dart
```
```dart
import 'package:flutter/material.dart';
import '../controller/cliente_controller.dart';
import '../model/cliente.dart';
import 'cliente_form_page.dart';

class ClienteListPage extends StatefulWidget {
  const ClienteListPage({super.key});

  @override
  State<ClienteListPage> createState() => _ClienteListPageState();
}

class _ClienteListPageState extends State<ClienteListPage> {
  final controller = ClienteController();

  List<Cliente> clientes = [];

  carregarClientes() async {
    clientes = await controller.listarClientes();

    setState(() {});
  }

  @override
  void initState() {
    super.initState();

    carregarClientes();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Clientes'),
      ),

      floatingActionButton: FloatingActionButton(
        onPressed: () async {
          final result = await Navigator.push(
            context,
            MaterialPageRoute(
              builder: (_) => const ClienteFormPage(),
            ),
          );

          if (result == true) {
            carregarClientes();
          }
        },
        child: const Icon(Icons.add),
      ),

      body: ListView.builder(
        itemCount: clientes.length,
        itemBuilder: (context, index) {
          final cliente = clientes[index];

          return ListTile(
            title: Text(cliente.nome),
            subtitle: Text(cliente.email),

            trailing: Row(
              mainAxisSize: MainAxisSize.min,
              children: [
                IconButton(
                  icon: const Icon(Icons.edit),
                  onPressed: () async {
                    final result = await Navigator.push(
                      context,
                      MaterialPageRoute(
                        builder: (_) => ClienteFormPage(
                          cliente: cliente,
                        ),
                      ),
                    );

                    if (result == true) {
                      carregarClientes();
                    }
                  },
                ),

                IconButton(
                  icon: const Icon(Icons.delete),
                  onPressed: () async {
                    await controller.deletarCliente(cliente.id!);

                    carregarClientes();
                  },
                ),
              ],
            ),
          );
        },
      ),
    );
  }
}
```

### Explicação da Tela
- ListView.builder
    - Cria uma lista de itens.
    - É possível criar um item para cada elemento da lista.
- FloatingActionButton
    - É um botão flutuante que aparece no canto superior direito da tela.
    - É possível adicionar um botão para adicionar novos clientes.
- Navigator.push()
    - Permite que você navegue para outra tela.
    - É possível passar dados para a tela que está sendo navegada.
- IconButton
    - É um botão que exibe um ícone.
    - É possível adicionar um botão para editar ou excluir clientes.


## Configurando o main.dart
```
lib/main.dart
```
```dart
import 'package:flutter/material.dart';
import 'view/cliente_list_page.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      title: 'CRUD Clientes',
      home: const ClienteListPage(),
    );
  }
}
```



