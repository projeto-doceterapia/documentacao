# doceterapia
🍬 Doce Terapia – Sistema de Controle de Estoque
Um sistema desenvolvido para a doceria Doce Terapia, com o objetivo de gerenciar o estoque de ingredientes de forma eficiente, segura e escalável.

📋 Funcionalidades
Cadastro de ingredientes com informações detalhadas (nome, quantidade, unidade de medida, validade).

Controle de entrada e saída de estoque.

Alertas de baixo estoque e vencimento próximo.

Relatórios de consumo e histórico de movimentações.

Interface amigável para funcionários da doceria.

🛠️ Tecnologias Utilizadas
Camada	Tecnologias
Frontend	HTML, CSS, JavaScript, React, Bootstrap
Backend	Node.js, Java 21, Spring Boot
Banco de Dados	MySQL
Infraestrutura	AWS (servidor em nuvem)
🚀 Como Executar o Projeto
Pré-requisitos
Node.js instalado

Java 21 instalado

MySQL configurado

Conta AWS para deploy

Passos
Clone o repositório:

bash
git clone https://github.com/seu-usuario/doce-terapia.git
Instale as dependências do frontend:

bash
cd frontend
npm install
Configure o backend (Spring Boot + Node.js):

Ajuste as credenciais do banco de dados no arquivo application.properties.

Execute:

bash
mvn spring-boot:run
Inicie o frontend:

bash
npm start
Acesse no navegador:

Código
http://localhost:3000
📂 Estrutura do Projeto
Código
doce-terapia/
├── frontend/        # React + Bootstrap
├── backend/         # Spring Boot + Node.js
├── database/        # Scripts MySQL
└── docs/            # Documentação
🌐 Deploy
O sistema está hospedado na AWS, garantindo escalabilidade e alta disponibilidade.

🤝 Contribuição
Contribuições são bem-vindas!

Faça um fork do projeto.

Crie uma branch para sua feature (git checkout -b minha-feature).

Commit suas alterações (git commit -m 'Adiciona minha feature').

Push para a branch (git push origin minha-feature).

Abra um Pull Request.
