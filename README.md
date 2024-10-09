# dio-expert-session-finance

## Pré Desenvolvimento

1. Vamos criar um projeto no Github chamado `dio-expert-session-finance`
    - Depois voltamos aqui para configurar mais nosso repositório. Por enquanto vamos ter apenas arquivos básicos
        - `.gitignore`
        - `README.md`

2. Depois do nosso projeto criado no Github, vamos criar um projeto `Go` básico. 
    - `$ mkdir dio-expert-session-finance`
    - `$ cd dio-expert-session-finance`
    - `$ go mod init github.com/marcopollivier/dio-expert-session-finance`
    
    Esses comandos vão criar dentro da pasta `dio-expert-session-finance` um arquivo `go.mod`. 
    E esse arquivo vai ser a base do nosso projeto. 
    
    Depois disso, você já pode abrir seu projeto na IDE de sua escolha
    - GoLand 
    - VSCode 
    - Vim 
    
 3. Agora que temos o nosso projeto funcionando corretamente, vamos criar um `Hello, World!` para 
 termos certeza que tudo está de acordo com o que esperávamos. 
     - Dentro da pasta `dio-expert-session-finance/cmd/server/` vamos criar o arquivo `main.go`
     
    ```go
    package main
    
    import "fmt"
    
    func main() {
        fmt.Print("Olá, Mundo!")
    }    
    ```

    Com isso já vemos que nosso ambiente está ok e funcional, mas ainda não é isso que queremos exatamente, correto? 

4. Mas precisamos evoluir nosso código para começar a tomar forma de uma API. 

    ```go
    package main
    
    import (
           "fmt"
           "net/http"
    )
    
    func main() {
           http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
                   fmt.Fprintf(w, "Olá. Bem vindo a minha página!")
           })
    
           http.ListenAndServe(":8080", nil)
    }
    ```
    `$ curl curl http://localhost:8080/`
    
5. Vamos começar a pensar no nosso modelo. Nós queremos criar um sistema pras nossas finanças pessoais. 
Para isso vamos pensar num modelo para trabalharmos. 

    5.1. Nosso modelo financeiro vai ser bem simples, mas flexível o suficiente para evoluirmos 
    no futuro
    
    - Titulo
    - Valor 
    - Tipo (ENTRADA, SAIDA)
    - Data
 
    5.2. Vamos pensar que, antes de mais nada, queremos retornar um JSON com esse modelo.
    
    ```go
    package main
    
    import (
    	"encoding/json"
    	"net/http"
    	"time"
    )
    
    func main() {
    	http.HandleFunc("/transactions", getTransactions)
    
    	_ = http.ListenAndServe(":8080", nil)
    }
    
    type Transaction struct {
    	Title     string
    	Amount    float32
    	Type      int //0. entrada 1. saida
    	CreatedAt time.Time
    }
    
    type Transactions []Transaction
    
    func getTransactions(w http.ResponseWriter, r *http.Request) {
    	if r.Method != "GET" {
    		w.WriteHeader(http.StatusMethodNotAllowed)
    		return
    	}
    
    	w.Header().Set("Content-type", "application/json")
    
    	layout := "2006-01-02T15:04:05"
    	salaryReceived, _ := time.Parse(layout, "2020-04-05T11:45:26")
    	paidElectricityBill, _ := time.Parse(layout, "2020-04-12T22:00:00")
    	var transactions = Transactions{
    		Transaction{
    			Title:     "Salário",
    			Amount:    1200.0,
    			Type:      0,
    			CreatedAt: salaryReceived,
    		},
    		Transaction{
    			Title:     "Conta de luz",
    			Amount:    100.0,
    			Type:      1,
    			CreatedAt: paidElectricityBill,
    		},
    	}
    
    	_ = json.NewEncoder(w).Encode(transactions)
    }
    ```

    ```go
    type Tction struct {
        Title     string    `json:"title"`
        Amount    float32   `json:"amount"`
        Type      int       `json:"type"` //0. entrada 1. saida
        CreatedAt time.Time `json:"created_at"`
    }
    ```

6. E agora vamos fazer um método de inserção (POST)
    
    ```go
    http.HandleFunc("/transactions/create", createATransaction)
   
    ...
   
    func createATransaction(w http.ResponseWriter, r *http.Request) {
    	if r.Method != "POST" {
    		w.WriteHeader(http.StatusMethodNotAllowed)
    		return
    	}
    
    	var res = Transactions{}
    	var body, _ = ioutil.ReadAll(r.Body)
    	_ = json.Unmarshal(body, &res)
    
    	fmt.Println(res)
    	fmt.Println(res.Title)
    	fmt.Println(res.Title)
    }
    ```
   
    ```json
    [
       {
           "title": "Salário",
           "amount": 1200,
           "type": 0,
           "created_at": "2020-04-05T11:45:26Z"
       }
    ]
    ```
   
   ```shell script
   $ curl -X POST 'http://localhost:8080/transactions/create' \
   -H 'Content-Type: application/json' \
   -d '[
           {
               "title": "Salário",
               "amount": 1200,
               "type": 0,
               "created_at": "2020-04-05T11:45:26Z"
           }
       ]'

Hora de refatorar. Vamos colocar em memória e isolar em arquivos e pacotes
Vamos começar a pensar em monitoramento? Então vamos criar um arquivo de Healthcheck
Go

package actuator

import (
    "encoding/json"
    "net/http"
)

func Health(responseWriter http.ResponseWriter, request *http.Request) {
    responseWriter.Header().Set("Content-Type", "application/json")

    profile := HealthBody{"alive"}

    returnBody, err := json.Marshal(profile)
    if err != nil {
        http.Error(responseWriter, err.Error(), http.StatusInternalServerError)
        return
    }

    _, err = responseWriter.Write(returnBody)
    if err != nil {
        http.Error(responseWriter, err.Error(), http.StatusInternalServerError)
        return
    }
}

type HealthBody struct {
    Status string
}
Código gerado por IA. Examine e use com cuidado. Mais informações em perguntas frequentes.
Vamos aproveitar já que criamos um util e escrever um teste unitário para ele
Go

package util

import (
	"testing"
)

func TestStringToDate(testing *testing.T) {
	var convertedTime = StringToTime("2019-02-12T10:00:00")

	if convertedTime.Year() != 2019 {
		testing.Errorf("Converter StringToDate is failed. Expected Year %v, got %v", 2019, convertedTime.Year())
	}

	if convertedTime.Month() != 2 {
		testing.Errorf("Converter StringToDate is failed. Expected Month %v, got %v", 2, convertedTime.Month())
	}

	if convertedTime.Hour() != 10 {
		testing.Errorf("Converter StringToDate is failed. Expected Hour %v, got %v", 10, convertedTime.Hour())
	}

}
Código gerado por IA. Examine e use com cuidado. Mais informações em perguntas frequentes.
Vamos começar a pensar em um pouco de qualidade de código também 10.1. Para fazer análise de código estática, vamos instalar a dependencia do lint
$ go get -u golang.org/x/lint/golint
10.2. E vamos executar nossos primeiros comandos relacionados
$ go test ./...
$ golint ./...
Vamos configurar o CircleCI
Vamos colocar métricas na nossa aplicação
$ go get github.com/prometheus/client_golang/prometheus
$ go get github.com/prometheus/client_golang/prometheus/promauto
$ go get github.com/prometheus/client_golang/prometheus/promhttp 

Para os passos seguintes, nós vamos fazer uma integração com um BD qualquer. Para isso, vamos subir uma imagem Docker do Postgres pra poder fazer o nosso teste. Vamos subir o BD via Docker Compose host: localhost user: postgres pass: postgres DB: diodb
version: "3"
services:
  postgres:
    image: postgres:9.6
    container_name: "postgres"
    environment:
      - POSTGRES_DB=diodb
      - POSTGRES_USER=postgres
      - TZ=GMT
    volumes:
      - "./data/postgres:/var/lib/postgresql/data"
    ports:
      - 5432:5432

prepare-tests:
   docker-compose -f .devops/postgres.yml up -d

14. Já com o banco acessível via Docker, vamos criar a base que utilizaremos no nosso teste:

    ```sql
    CREATE TABLE transactions (
        id SERIAL PRIMARY KEY,
        title VARCHAR(100),
        amount DECIMAL,
        type SMALLINT,
        installment SMALLINT,
        created_at TIMESTAMP 
    );
    
    INSERT INTO transactions (title, amount, type, installment, created_at)
    VALUES ('Freela', 100.0, 0, 1, '2020-04-10 04:05:06'); 
   
    SELECT * FROM transactions;
    ```

15. Agora com a estrutura de banco criada, vamos fazer as alterações necessárias no código. Primeiro, baixe a dependência do driver do Postgres:

    Lista de SQLDrivers disponíveis 

    Execute o seguinte comando dentro da pasta do projeto:
    
    ```shell
    $ go get -u github.com/lib/pq
    ```

16. Código para manipular as informações do banco:

    ```go
    package postgres
    
    import (
        "database/sql"
        "fmt"
        "github.com/marcopollivier/dio-expert-session-pre-class/model/transaction"
        _ "github.com/lib/pq"
    )
    
    const (
        host     = "localhost"
        port     = 5432
        user     = "postgres"
        password = "postgres"
        dbname   = "diodb"
    )
    
    func connect() *sql.DB {
        psqlInfo := fmt.Sprintf("host=%s port=%d user=%s password=%s dbname=%s sslmode=disable", host, port, user, password, dbname)
        db, err := sql.Open("postgres", psqlInfo)
        if err != nil {
            panic(err)
        }
        return db
    }
    
    func Create(transaction transaction.Transaction) int {
        db := connect()
        defer db.Close()
    
        sqlStatement := `INSERT INTO transactions (title, amount, type, installment, created_at)
                         VALUES ($1, $2, $3, $4, $5) RETURNING id;`
    
        var id int
        err := db.QueryRow(sqlStatement, transaction.Title, transaction.Amount, transaction.Type, transaction.Installment, transaction.CreatedAt).Scan(&id)
        if err != nil {
            panic(err)
        }
        fmt.Println("New record ID is:", id)
    
        return id
    }
    
    func FetchAll() transaction.Transactions {
        db := connect()
        defer db.Close()
    
        rows, err := db.Query("SELECT title, amount, type, installment, created_at FROM transactions")
        if err != nil {
            panic(err)
        }
        defer rows.Close()
    
        var transactions transaction.Transactions
        for rows.Next() {
            var t transaction.Transaction
            err := rows.Scan(&t.Title, &t.Amount, &t.Type, &t.Installment, &t.CreatedAt)
            if err != nil {
                panic(err)
            }
            transactions = append(transactions, t)
        }
    
        return transactions
    }
    
    func main() {
        fmt.Println(FetchAll())
    }
    ```

## Redes Sociais

Para mais informações, entre em contato comigo através das minhas redes sociais:

## Contato
Para mais informações, entre em contato através das minhas redes sociais:

[![LinkedIn](https://img.shields.io/badge/LinkedIn-0077B5?style=for-the-badge&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/rafaelgoncalvesmiguel/)
[![Instagram](https://img.shields.io/badge/Instagram-E4405F?style=for-the-badge&logo=instagram&logoColor=white)](https://www.instagram.com/rafagoncalves.18/)
