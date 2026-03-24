# 🍬 Doce Terapia – Sistema de Controle de Estoque
Um sistema desenvolvido para a doceria Doce Terapia, com o objetivo de gerenciar o estoque de ingredientes de forma eficiente, segura e escalável.

📋 Funcionalidades<br>
Cadastro de ingredientes com informações detalhadas (nome, quantidade, unidade de medida, validade).

Controle de entrada e saída de estoque.

Alertas de baixo estoque e vencimento próximo.

Relatórios de consumo e histórico de movimentações.

Interface amigável para funcionários da doceria.

🛠️ Tecnologias Utilizadas<br>
Camada	Tecnologias<br>
Frontend	HTML, CSS, JavaScript, React, Bootstrap<br>
Backend	Node.js, Java 21, Spring Boot<br>
Banco de Dados	MySQL<br>
Infraestrutura	AWS (servidor em nuvem)<br>

🚀 Como Executar o Projeto<br>
Pré-requisitos<br>
Node.js instalado<br>

Java 21 instalado

MySQL configurado

Conta AWS para deploy

Passos<br>
Clone o repositório:

bash<br>
git clone https://github.com/seu-usuario/doce-terapia.git<br>
Instale as dependências do frontend:<br>

bash<br>
cd frontend<br>
npm install<br>
Configure o backend (Spring Boot + Node.js):

Ajuste as credenciais do banco de dados no arquivo application.properties.

Execute:

bash<br>
mvn spring-boot:run<br>
Inicie o frontend:

bash<br>
npm start<br>
Acesse no navegador:<br>

http://localhost:3000

📂 Estrutura do Projeto<br>
Código<br>
doce-terapia/<br>
├── prototipo/ #Prototipo funcional do MVP <br>
├── front-end/ # React + Bootstrap<br>
├── back-end/ # Spring Boot + Node.js<br>
├── banco-de-dados/ # Scripts MySQL<br>
└── documentacao/ # Documentação<br>

🌐 Deploy<br>
O sistema está hospedado na AWS, garantindo escalabilidade e alta disponibilidade.
