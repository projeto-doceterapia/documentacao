# 6. Premissas

Premissas são condições que o sistema assume como verdadeiras. Se uma delas deixar de valer, o desenho da solução precisa ser revisto.

| ID | Premissa | Impacto se não se confirmar |
|---|---|---|
| P01 | A usuária tem acesso a computador (e celular) com internet. | O sistema web não pode ser usado |
| P02 | A usuária tem pouca familiaridade com sistemas digitais, então a curva de aprendizado precisa ser mínima. | Rejeição da ferramenta |
| P03 | O uso será rápido e encaixado na rotina de produção. | Telas longas serão abandonadas |
| P04 | Os pedidos continuam chegando pelo WhatsApp, que segue como canal principal. O sistema é apoio organizacional e funciona em paralelo. | Seria preciso redesenhar a captação de pedidos |
| P05 | A usuária registra os dados manualmente e de forma correta. A confiabilidade das métricas depende disso. | Estoque e métricas inconsistentes |
| P06 | A solução precisa ser simples e fácil de usar, sem termos técnicos. | Baixa adoção |
| P07 | A confeiteira fica disponível para validar as telas e os fluxos durante o desenvolvimento. | Requisitos não validados |
| P08 | A operação tem uma única usuária administradora. | Seria necessário controle de perfis e permissões |
| P09 | O volume é baixo, na ordem de dezenas de mensagens e pedidos por dia (cerca de 50 mensagens por dia). | Infraestrutura de fila e cache seria necessária |
