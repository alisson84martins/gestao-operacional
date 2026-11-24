# Gestão Operacional de Garagem

**Pátio, portaria, manutenção, ocorrências e fiscalização num só sistema, registrados uma única vez por quem vive o problema.**

> 🔒 **Vitrine do projeto.** O código-fonte é privado porque o sistema está em testes reais numa operação de transporte. Aqui estão o problema, as decisões e a arquitetura. O código pode ser apresentado sob demanda.

## O problema

Numa garagem de ônibus urbanos, a mesma informação é escrita várias vezes. A ocorrência é preenchida no papel, depois passada a limpo, depois transformada em mensagem, e as fotos seguem por WhatsApp.
A posição dos carros no pátio vive em pranchetas, e o estado do veículo na saída não fica registrado em lugar nenhum.

Cada etapa manual é um ponto onde a informação se perde, atrasa ou chega diferente para cada setor.

## A proposta

Um sistema web, que abre no celular, em que **cada informação é registrada uma vez, por quem está no local**, e fica disponível para todos os setores que dependem dela.

## Módulos

| Módulo | Para quem | O que resolve |
|---|---|---|
| 🅿️ **Pátio** | Operador de pátio e coordenação | Onde cada carro está, em que fila e em que ordem sai, com importação do planejamento do dia |
| 🚧 **Portaria** | Controle de acesso | Entrada e saída de veículos, avaria marcada num mapa do ônibus e recolhida fora do padrão |
| 🔧 **Manutenção** | Mecânica e funilaria | Fila de recolhidas e avarias, com baixa do conserto e liberação do carro |
| 📋 **Ocorrências** | Coordenação | Registro completo, mensagem padronizada para o setor de sinistros, fotos, documentos e busca por veículo, linha ou matrícula |
| 📱 **Ocorrência interna** | Motoristas e cobradores | A ficha de ocorrência no celular, com foto, que entra numa fila para a coordenação avaliar |
| 🧭 **Fiscalização** | Fiscais em campo | Turno do fiscal, registro de partidas no ponto e mensagem de fechamento montada automaticamente, com indicador de cumprimento de viagens por linha |
| 📅 **Escala de fiscais** | Coordenação | Escala semanal fora da planilha |
| 📊 **Painel gerencial** | Gerência | Visão só de leitura do que Portaria e Pátio registram, com histórico e exportação |
| ⚙️ **Administração** | Administrador | Cadastro central de pessoas, funções e permissões |

Os módulos se conectam pelo **prefixo do veículo** e pela **matrícula da pessoa**: a ocorrência de um carro pode ser cruzada com a avaria registrada na saída e com o histórico de manutenção.

## Testes em operação real (jul–nov/2026)

O desenho de cada módulo saiu de testes na rotina de uma garagem em São Paulo:

| Frente | Resultado |
|---|---|
| **Ocorrências** | 85 ocorrências registradas por 6 coordenadores, cerca de 25 por mês. Busca de todas as ocorrências de um carro em menos de 1 segundo |
| **Pátio** | Alocação dos carros nas filas, ordem de saída e importação do planejamento diário |
| **Portaria** | Entrada e saída de veículos, avaria na saída da frota e recolhida fora do padrão |

Os ajustes que vieram de cada teste foram documentados nas migrações do banco, com o porquê de cada mudança.

## Arquitetura

```
Navegador / celular ──HTTPS──> nginx ──> API (FastAPI) ──> PostgreSQL
                                 │                │
                          telas estáticas   anexos fora da web
```

| Camada | Tecnologia |
|---|---|
| **Telas** | HTML, CSS e JavaScript puro (ES modules), sem framework e sem build. Abre em qualquer navegador, inclusive no celular |
| **API** | Python 3.11, FastAPI, SQLAlchemy 2, Pydantic |
| **Banco** | PostgreSQL organizado em schemas por módulo, evoluído por migrações numeradas e versionadas |
| **Servidor** | Ubuntu, nginx com HTTPS (TLS 1.2 e 1.3) e systemd. Só o nginx fica exposto; API e banco escutam apenas dentro do servidor |
| **Qualidade** | Testes automatizados com pytest e testes externos contra o servidor |

## Segurança e LGPD

- Login por matrícula, senha em **bcrypt** e token **JWT** com validade de um turno
- Permissão por **função e recurso** (ler ou escrever), com exceção por pessoa (RBAC)
- Ocorrência só pode ser alterada por quem registrou; depois de finalizada, toda alteração vai para um **histórico que não se apaga**
- Anexos conferidos pelo **conteúdo real do arquivo**, gravados fora da pasta pública e entregues só a quem tem permissão
- **Trilha de auditoria** de acessos e de leitura de ocorrências
- Bloqueio de tentativas repetidas de senha
- Nenhuma senha, chave ou configuração de produção versionada

## Status

Protótipo funcional em avaliação técnica, para estudar como os módulos podem ser incorporados aos sistemas da operação.

---

**Alisson Martins** · Idealização, modelagem e desenvolvimento
[LinkedIn](https://www.linkedin.com/in/alisson-martins-2a350238a) · [E-mail](mailto:alisson84martins@gmail.com)
