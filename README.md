# crud
crud database
from persistent import Persistent
from datetime import datetime, timedelta
from ZODB import FileStorage, DB
import transaction


class Pessoa(Persistent):
    def __init__(self, id, nome, cpf, email):
        self.id = id
        self.nome = nome
        self.cpf = cpf
        self.email = email

class Usuario(Pessoa):
    def __init__(self, id, nome, cpf, email, numero_cartao):
        super().__init__(id, nome, cpf, email)
        self.numero_cartao = numero_cartao
        self.ativo = True

class Livro(Persistent):
    def __init__(self, id, titulo, autor, isbn):
        self.id = id
        self.titulo = titulo
        self.autor = autor
        self.isbn = isbn
        self.disponivel = True

class Emprestimo(Persistent):
    def __init__(self, id, usuario, livro):
        self.id = id
        self.usuario = usuario
        self.livro = livro
        self.data_emprestimo = datetime.now()
        self.data_devolucao = datetime.now() + timedelta(days=14)
        self.devolvido = False
        self.livro.disponivel = False
        self._p_changed = True  

    def devolver(self):
        self.devolvido = True
        self.livro.disponivel = True
        self._p_changed = True 

storage = FileStorage.FileStorage('biblioteca.fs')
db = DB(storage)
connection = db.open()
root = connection.root()


if not hasattr(root, 'usuarios'):
    root.usuarios = {}
if not hasattr(root, 'livros'):
    root.livros = {}
if not hasattr(root, 'emprestimos'):
    root.emprestimos = {}

def criar_usuario(id, nome, cpf, email, numero_cartao):
    usuario = Usuario(id, nome, cpf, email, numero_cartao)
    root.usuarios[id] = usuario
    transaction.commit()
    print(f"Usuário {nome} criado com sucesso!")

def listar_usuarios():
    for id, usuario in root.usuarios.items():
        print(f"ID: {id}, Nome: {usuario.nome}, Email: {usuario.email}")

def atualizar_usuario(id, novo_nome, novo_email):
    if id in root.usuarios:
        usuario = root.usuarios[id]
        usuario.nome = novo_nome
        usuario.email = novo_email
        usuario._p_changed = True  
        transaction.commit()
        print(f"Usuário {novo_nome} atualizado com sucesso!")
    else:
        print("Usuário não encontrado.")

def deletar_usuario(id):
    if id in root.usuarios:
        del root.usuarios[id]
        transaction.commit()
        print("Usuário deletado com sucesso!")
    else:
        print("Usuário não encontrado.")


def criar_livro(id, titulo, autor, isbn):
    livro = Livro(id, titulo, autor, isbn)
    root.livros[id] = livro
    transaction.commit()
    print(f"Livro '{titulo}' criado com sucesso!")

def listar_livros():
    for id, livro in root.livros.items():
        print(f"ID: {id}, Título: {livro.titulo}, Autor: {livro.autor}, Disponível: {livro.disponivel}")

def criar_emprestimo(id, usuario_id, livro_id):
    if usuario_id in root.usuarios and livro_id in root.livros:
        usuario = root.usuarios[usuario_id]
        livro = root.livros[livro_id]
        emprestimo = Emprestimo(id, usuario, livro)
        root.emprestimos[id] = emprestimo
        transaction.commit()
        print(f"Empréstimo do livro '{livro.titulo}' para {usuario.nome} criado com sucesso!")
    else:
        print("Usuário ou Livro não encontrado.")

def devolver_emprestimo(id):
    if id in root.emprestimos:
        emprestimo = root.emprestimos[id]
        emprestimo.devolver()
        transaction.commit()
        print(f"Livro '{emprestimo.livro.titulo}' devolvido com sucesso!")
    else:
        print("Empréstimo não encontrado.")

def menu():
    while True:
        print("\n--- Menu da Biblioteca ---")
        print("1. Usuários")
        print("2. Livros")
        print("3. Empréstimos")
        print("4. Sair")
        opcao_principal = input("Escolha uma opção: ")

        if opcao_principal == "1":
            menu_usuarios()
        elif opcao_principal == "2":
            menu_livros()
        elif opcao_principal == "3":
            menu_emprestimos()
        elif opcao_principal == "4":
            print("Saindo...")
            break
        else:
            print("Opção inválida. Tente novamente.")

def menu_usuarios():
    while True:
        print("\n--- Menu de Usuários ---")
        print("1. Criar Usuário")
        print("2. Listar Usuários")
        print("3. Atualizar Usuário")
        print("4. Deletar Usuário")
        print("5. Voltar")
        opcao = input("Escolha uma opção: ")

        if opcao == "1":
            id = input("ID do usuário: ")
            nome = input("Nome: ")
            cpf = input("CPF: ")
            email = input("Email: ")
            numero_cartao = input("Número do Cartão: ")
            criar_usuario(id, nome, cpf, email, numero_cartao)
        elif opcao == "2":
            listar_usuarios()
        elif opcao == "3":
            id = input("ID do usuário a ser atualizado: ")
            novo_nome = input("Novo Nome: ")
            novo_email = input("Novo Email: ")
            atualizar_usuario(id, novo_nome, novo_email)
        elif opcao == "4":
            id = input("ID do usuário a ser deletado: ")
            deletar_usuario(id)
        elif opcao == "5":
            break
        else:
            print("Opção inválida. Tente novamente.")

def menu_livros():
    while True:
        print("\n--- Menu de Livros ---")
        print("1. Criar Livro")
        print("2. Listar Livros")
        print("3. Voltar")
        opcao = input("Escolha uma opção: ")

        if opcao == "1":
            id = input("ID do livro: ")
            titulo = input("Título: ")
            autor = input("Autor: ")
            isbn = input("ISBN: ")
            criar_livro(id, titulo, autor, isbn)
        elif opcao == "2":
            listar_livros()
        elif opcao == "3":
            break
        else:
            print("Opção inválida. Tente novamente.")

def menu_emprestimos():
    while True:
        print("\n--- Menu de Empréstimos ---")
        print("1. Criar Empréstimo")
        print("2. Devolver Empréstimo")
        print("3. Voltar")
        opcao = input("Escolha uma opção: ")

        if opcao == "1":
            id = input("ID do empréstimo: ")
            usuario_id = input("ID do usuário: ")
            livro_id = input("ID do livro: ")
            criar_emprestimo(id, usuario_id, livro_id)
        elif opcao == "2":
            id = input("ID do empréstimo a ser devolvido: ")
            devolver_emprestimo(id)
        elif opcao == "3":
            break
        else:
            print("Opção inválida. Tente novamente.")

if __name__ == "__main__":
    menu()
