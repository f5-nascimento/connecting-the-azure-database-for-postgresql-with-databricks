# Conectando o Azure Database for PostgreSQL com Azure Databricks
Tutorial passo a passo de como criar um workspace no Azure Databricks e conectá-lo a o Azure Database for PostgreSQL

## ✅ Pré-requisitos

- Conta no [Azure](https://portal.azure.com/)
---

## 🧭 Etapas do tutorial

### 1. Acesse o Portal do Azure
- Acesse: [https://portal.azure.com](https://portal.azure.com)
- Faça login com sua conta Microsoft.

---

### 2. Crie um Servidor de Banco de Dados PostgreSQL

<img width="1493" height="615" alt="postgre" src="https://github.com/user-attachments/assets/8d57dce8-b3b2-4f5a-bd81-0f1d14e0fdb4" />

- No campo de busca do portal, digite **Azure Database for PostgreSQL**
- Clique no ícone conforme a imagem acima.
- Você será direcionado a uma tela para *Novo servidor flexível do Banco de Dados do Azure para PostgreSQL*
- **Assinatura**: Selecione sua assinatura ativa (ex: `Azure for Students` ou `Azure subscription 1`)
- **Grupo de recursos**: Selecione um existente (ex: `lab-sql`) ou clique em **Criar novo**
- **Nome do servidor:** Defina um nome exclusivo para o servidor (ex: `server-pgsql255`)
- **Região**: Escolha uma região próxima ou mais barata (ex: `West US 3`)
- **Tipo de carga de trabalho**: selecione `Desenvolvimento`
- **Método de autenticação**: Selecione `PostgreSQL somente autenticação`
- **Logon do administrador**: Insira um nome de administrador (ex: `adminpgsql`) 
- **Senha**: Insira a senha de administrador (ex: `pgServer123`) 
- **Confirmar senha**: Repita a senha informada
> Sua senha não pode conter todo ou parte do nome de logon. Parte do nome de logon é definida como três ou mais caracteres alfanuméricos consecutivos.
- Clique em **Avançar: Rede >**
- Marque a opção *Adicionar a regra de firewall ao endereço de IP atual*
- Clique em **Revisar e criar**
- Clique em **Criar**
> Permitir o acesso público de qualquer serviço do Azure de dentro do Azure para esse servidor
---

### 3. Crie um Banco de Dados PostgreSQL no Servidor

<img width="1876" height="702" alt="bd" src="https://github.com/user-attachments/assets/382e449f-96f0-40c8-aabc-4e07878626f9" />

- Após a implantação, clique em **Ir para o recursos**.
- No menu lateral esquerdo da nova tela, clique em **Configurações**.
- Selecione **Banco de dados**
- Clique em **+ Adicionar**
- Em **Criar banco de dados**, defina o nome como: **regencia**
- Clique em **Salvar**


## 4. Criando o Workspace no Azure Databricks

<img width="1517" height="355" alt="azdatabricks" src="https://github.com/user-attachments/assets/3fe73abf-fb05-4bc8-8b6a-5ea38194affc" />

- No campo de busca do portal, digite **Azure Databricks**
- Clique no ícone conforme a imagem acima.
- Após o redirecionamento para uma nova tela, clique em **Criar**
- Na tela **Criar um workspace do Azure Databricks**, preencha os passos:
- **Assinatura**: Selecione sua assinatura ativa (ex: `Azure for Students` ou `Azure subscription 1`)
- **Grupo de recursos**: Selecione um existente (ex: `lab-databricks`) ou clique em **Criar novo**
- **Nome do Workspace**: Defina um nome exclusivo para o workspace (ex: `work-databricks`)
- **Região** (ex: West US 3)
- **Tipo de Preço**: Selecione a opção `Avaliação`
- **Nome do Grupo de Recursos Gerenciados**: deixe em branco (será preenchido automaticamente)
- Clique em **Revisar + Criar**
- Após a validação, clique em **Criar**

---

## 5. Acessando o Azure Databricks

<img width="1877" height="747" alt="workpace" src="https://github.com/user-attachments/assets/6da50ec1-07bb-4270-8d83-79be594434c7" />

- Após a implantação, clique em **Ir para o recurso**
- Clique em **Iniciar workspace**
- Você será redirecionado para o ambiente Databricks

---

## 6. Criando um notebook

<img width="1782" height="637" alt="notebooks" src="https://github.com/user-attachments/assets/a296c932-e652-44c9-8f0c-61b843041920" />

- No menu lateral cliquem em **+ Novo**
- Depois clique em **Notebook**
- Copie o código abaixo no notebook:
  
```python
import psycopg2

# Configurações do banco
host = ""
database = ""
user = ""  
password = ""
port = 5432

try:
    # Testa conexão
    conn = psycopg2.connect(
        host=host,
        database=database,
        user=user,
        password=password,
        port=port,
        sslmode='require'
    )
    print("✅ Conexão bem-sucedida!")

    cur = conn.cursor()

    while True:
        print("\n=== MENU DE OPERAÇÕES SQL ===")
        print("1 - Criar tabela 'usuarios'")
        print("2 - SELECT")
        print("3 - INSERT")
        print("4 - UPDATE")
        print("5 - DELETE")
        print("6 - Sair")

        opcao = input("Digite o número da operação desejada: ")

        if opcao == '1':
            cur.execute("""
                CREATE TABLE IF NOT EXISTS usuarios (
                    id SERIAL PRIMARY KEY,
                    nome VARCHAR(100)
                );
            """)
            conn.commit()
            print("✅ Tabela criada com sucesso.")

        elif opcao == '2':
            cur.execute("SELECT * FROM usuarios;")
            resultados = cur.fetchall()
            print("📋 Resultados:")
            for linha in resultados:
                print(linha)

        elif opcao == '3':
            nome = input("Digite o nome para inserir: ")
            cur.execute("INSERT INTO usuarios (nome) VALUES (%s);", (nome,))
            conn.commit()
            print("✅ Registro inserido.")

        elif opcao == '4':
            id_atual = input("Digite o id do usuário para atualizar: ")
            novo_nome = input("Digite o novo nome: ")
            cur.execute("UPDATE usuarios SET nome = %s WHERE id = %s;", (novo_nome, id_atual))
            conn.commit()
            print("✅ Registro atualizado.")

        elif opcao == '5':
            id_del = input("Digite o id do usuário para deletar: ")
            cur.execute("DELETE FROM usuarios WHERE id = %s;", (id_del,))
            conn.commit()
            print("✅ Registro deletado.")

        elif opcao == '6':
            print("👋 Encerrando o programa...")
            break

        else:
            print("❌ Opção inválida, tente novamente.")

    cur.close()
    conn.close()

except psycopg2.OperationalError as e:
    print("❌ Falha na conexão:")
    print(e)

except Exception as e:
    print("❌ Outro erro:")
    print(e)

```

---

## 7. Coletando os dados do PostgreSQL

<img width="1866" height="467" alt="cred" src="https://github.com/user-attachments/assets/baf83387-fd28-4126-a99a-0ea59c9fb64f" />

- Acesse a tela principal do seu servidor PostgreSQL  
> Extraia as seguintes informações para conexão:

| Parâmetro | Informações do Servidor PostgreSQL                             | Exemplo extraído                             |
|-----------|---------------------------------------------|---------------------------------------------|
| host      | `Ponto de Extremidade` |`server-pgsql255.postgres.database.azure.com` |
| database  | `nome do banco`                           |`regencia`                           |
| user      | `Logon do administrador`                              |`adminpgsql`                              |
| password  | `senha definida na criação do servidor`                          |`pgServer123`                          |
| port      | `5432`        |`5432`        |


## 8. Executando o notebook

<img width="1787" height="650" alt="roda" src="https://github.com/user-attachments/assets/cf74cb14-723a-4fbc-ba99-f76d8572b086" />


- Altere o código com os dados da sua URL, siga o exemplo abaixo

```python
# Configurações do banco
host = "server-pgsql255.postgres.database.azure.com"
database = "regencia"
user = "adminpgsql"  
password = "pgServer123"
port = 5432
```
- Para executar o código clique em **Ligar**
- Selecione **Serveless**
- Clique em **Executar célula**
---


