# Plataforma-de-Cursos-PIM-
Projeto de faculdade feito como Projeto de Integração Multidisciplinar (PIM), o tema é uma Plataforma de Cursos para uma ONG.

import json
import getpass
import msvcrt
import os
import hashlib
import statistics 
from statistics import mean, median, mode  

cadastro_usuario = "usuarios.json"
arquivo_acessos = "acessos.json"

def gerar_hash(senha):
    return hashlib.sha256(senha.encode()).hexdigest()

def usuario_existe(campo, valor):
    if not os.path.exists(cadastro_usuario):
        return False

    with open(cadastro_usuario, "r", encoding="utf-8") as arquivo:
        try:
            usuarios = json.load(arquivo)
        except json.JSONDecodeError:
            return False

    if isinstance(usuarios, list):
        return any(usuario.get(campo) == valor for usuario in usuarios)
    elif isinstance(usuarios, dict):
        return usuarios.get(campo) == valor
    return False

def primeiro_acesso():
    print("Cadastro de Usuário:")

    nome = input("\nDigite o seu nome: ")
    if usuario_existe("nome", nome):
        print("\nNome já cadastrado. Tente outro.")
        print("\nPressione qualquer tecla para continuar...")
        msvcrt.getch()
        os.system('cls') 
        return

    while True:
        try:
            idade = int(input("Digite a sua idade: "))
            break
        except ValueError:
            os.system('cls') 
            print("Valor inválido. Por favor, digite um número inteiro.\n")
         
    email = input("Digite o seu email: ")
    if usuario_existe("email", email):
        print("Email já cadastrado. Tente outro.")
        print("\nPressione qualquer tecla para continuar...")
        msvcrt.getch()
        os.system('cls') 
        return

    while True:
        senha = getpass.getpass("Digite a nova senha: ")
        confirmar = getpass.getpass("Confirme a senha: ")

        if senha != confirmar:
            os.system('cls') 
            print("As senhas não coincidem. Tente novamente.\n")
             
        elif senha.strip() == "":
            os.system('cls') 
            print("Senha não pode ser vazia. Tente novamente.\n")
            
        else:
            os.system('cls') 
            print("\nSenha cadastrada com sucesso!")
            break
    
    senha_hash = gerar_hash(senha)
            
    usuario = {
        "nome": nome,
        "idade": idade,
        "email": email,
        "senha": senha_hash
    }

    usuarios = []
    if os.path.exists(cadastro_usuario):
        with open(cadastro_usuario, "r", encoding="utf-8") as arquivo:
            try:
                usuarios = json.load(arquivo)
            except json.JSONDecodeError:
                usuarios = []

    if not isinstance(usuarios, list):
        usuarios = [usuarios]

    usuarios.append(usuario)

    with open(cadastro_usuario, "w", encoding="utf-8") as arquivo:
        json.dump(usuarios, arquivo, ensure_ascii=False, indent=4)

    print("\nCadastro realizado com sucesso!")
    print("\nPressione qualquer tecla para continuar...")
    msvcrt.getch()
    os.system('cls')    

def login_usuario():
    print("Login de Usuário:")
    nome_usuario = input("\nDigite o nome de usuário: ")

    if not os.path.exists(cadastro_usuario):
        print("Nenhum usuário cadastrado.")
        return

    with open(cadastro_usuario, "r", encoding="utf-8") as arquivo:
        try:
            usuarios = json.load(arquivo)
        except json.JSONDecodeError:
            print("Erro ao ler o arquivo de usuários.")
            return

    if not isinstance(usuarios, list):
        usuarios = [usuarios]

    for usuario in usuarios:
        if usuario["nome"] == nome_usuario:
            senha = getpass.getpass("Digite a senha: ")
            senha_hash = gerar_hash(senha)
            if usuario["senha"] == senha_hash:
                os.system('cls')
                print("Login bem sucedido!")
                registrar_acesso(usuario)
                print("\nPressione qualquer tecla para continuar...")
                msvcrt.getch()
                os.system('cls')
                opcoes_login()  
                return
            else:
                os.system('cls')
                print("Senha incorreta.")
                print("\nPressione qualquer tecla para continuar...")
                msvcrt.getch()
                
                os.system('cls')
                return

    print("Usuário não encontrado.")
    print("\nPressione qualquer tecla para continuar...")
    msvcrt.getch()
    os.system('cls')

def opcoes_login():
    while True:
        print("\n1. Conteúdo de estudo")
        print("2. Exibir estatísticas")
        print("3. Sair")
        opcao = input("\nEscolha uma opção: ")

        if opcao == "1":
            os.system('cls')
            conteudo_estudo()   
        elif opcao == "2":
            os.system('cls')
            estatisticas_usuario()
        elif opcao == "3":
            os.system('cls')
            print("Saindo...")
            exit()           
        else:
            os.system('cls')
            print("Opção inválida.")
            return
        
def estatisticas_usuario():
    while True:        
        print("1. Estatísticas de acesso dos usuários.")
        print("2. Estatísticas de idade dos usuários.")
        print("\n3. Voltar ao menu anterior")
        print("4. Sair")
        opcao = input("\nDigite qual estatística deseja visualizar ou opção desejada: ")
                    
        if opcao == "1":
            os.system('cls')
            estatisticas_acessogeral()
        elif opcao == "2":
            os.system('cls')
            estatisticas_idade()
        elif opcao == "3":
            os.system('cls')
            opcoes_login()
            exit()
        elif opcao == "4":
            os.system('cls')
            print("Saindo...")
            exit()
        else:
            os.system('cls')
            print("Opção inválida.")
            return                        
                                    
def carregar_acessos():
    if os.path.exists(arquivo_acessos):
        with open(arquivo_acessos, "r", encoding="utf-8") as f:
            return json.load(f)
    return {}

def salvar_acessos(acessos):
    with open(arquivo_acessos, "w", encoding="utf-8") as f:
        json.dump(acessos, f, indent=4, ensure_ascii=False)

def registrar_acesso(usuario):
    acessos = carregar_acessos()
    nome = usuario["nome"]

    if nome in acessos:
        acessos[nome] += 1
    else:
        acessos[nome] = 1

    salvar_acessos(acessos)

def estatisticas_acessogeral():
    acessos = carregar_acessos()
    if not acessos:
        print("Nenhum acesso registrado.")
        return

    print("Estatísticas gerais de acesso:")
    
    quantidades = list(acessos.values())
    
    media = statistics.mean(quantidades)
    mediana = statistics.median(quantidades)
    try:
        moda = statistics.mode(quantidades)
    except statistics.StatisticsError:
        moda = "Sem moda (valores igualmente frequentes)"

    print(f"\nMédia de acessos: {media:.2f}")
    print(f"Mediana de acessos: {mediana}")
    print(f"Moda de acessos: {moda}")

    print("\nEstatísticas por usuário:\n")
    for usuario, quantidade in acessos.items():
        print(f"{usuario}: {quantidade} acesso(s)")

    print("\nPressione qualquer tecla para continuar...")
    msvcrt.getch()
    os.system('cls')

def estatisticas_idade():
           
    with open('usuarios.json', 'r', encoding='utf-8') as arquivo:
        usuarios = json.load(arquivo)

    idades = [usuario['idade'] for usuario in usuarios]

    media = mean(idades)
    mediana = median(idades)
    try:
        moda = mode(idades)
    except:
        moda = "Sem moda (valores repetem com mesma frequência)"

    print(f"Média: {media}")
    print(f"Mediana: {mediana}")
    print(f"Moda: {moda}")
    print("\nPressione qualquer tecla para continuar...")
    msvcrt.getch()
    os.system('cls')
     
def conteudo_estudo():
    while True:
        print("\n1. Importância da inclusão digital")
        print("2. Evolução do computador ")
        print("3. História da internet")
        print("4. Pensamento Computacional ")
        print("5. Programação básica com Python")
        print("\n6. Voltar ao menu anterior")
        print("7. Sair")
        opcao = input("\nEscolha o conteúdo de estudo ou opção desejada: ")
        
        if opcao == "1":
            os.system('cls')
            inclusao_digital()
        elif opcao == "2":
            os.system('cls')
            evolucao_computador()
        elif opcao == "3":
            os.system('cls')
            historia_internet()
        elif opcao == "4":
            os.system('cls')
            pensamento_computacional()
        elif opcao == "5":
            os.system('cls')
            programacao_python()
        elif opcao == "6":
            os.system('cls')
            opcoes_login()
        elif opcao == "7":
            os.system('cls')
            print("Saindo...")
            exit()
        else:
            print(" Opção inválida.")
            print("\nPressione qualquer tecla para continuar...")
            msvcrt.getch()
            os.system('cls')

def inclusao_digital():
    with open('inclusao_estudo.txt', 'r', encoding='utf-8') as arquivo:
        inclusao_estudo = arquivo.read()
        print(inclusao_estudo)
        print("Pressione qualquer tecla para voltar ao menu anterior...")
        msvcrt.getch()
        os.system('cls')
        
def evolucao_computador():
    with open('evolucao_computador.txt', 'r', encoding='utf-8') as arquivo:
        evolucao_computador = arquivo.read()
        print(evolucao_computador)
        print("Pressione qualquer tecla para voltar ao menu anterior...")
        msvcrt.getch()
        os.system('cls')

def historia_internet():
    with open('historia_internet.txt', 'r', encoding='utf-8') as arquivo:
        historia_internet = arquivo.read()
        print(historia_internet)
        print("Pressione qualquer tecla para voltar ao menu anterior...")
        msvcrt.getch()
        os.system('cls')

def pensamento_computacional():
     with open('pensamento_computacional.txt', 'r', encoding='utf-8') as arquivo:
        pensamento_computacional = arquivo.read()
        print(pensamento_computacional)
        print("Pressione qualquer tecla para voltar ao menu anterior...")
        msvcrt.getch()
        os.system('cls')

def programacao_python():
    with open('programacao_python.txt', 'r', encoding='utf-8') as arquivo:
        programacao_python = arquivo.read()
        print(programacao_python)
        print("Pressione qualquer tecla para voltar ao menu anterior...")
        msvcrt.getch()
        os.system('cls')

def menu():
    while True:
        print("\n1. Primeiro acesso")
        print("2. Fazer Login")
        print("3. Sair")
        opcao = input("\n Escolha uma opção: ")

        if opcao == "1":
            os.system('cls')
            primeiro_acesso()
        elif opcao == "2":
            os.system('cls')
            login_usuario()
        elif opcao == "3":
            os.system('cls')
            print("Saindo...")
            exit()
            
        else:
            print("Opção inválida.")
            print("\nPressione qualquer tecla para continuar...")
            msvcrt.getch()
            os.system('cls')

menu()
