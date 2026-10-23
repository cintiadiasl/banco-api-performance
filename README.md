# Banco API Performance

Projeto de testes de performance de APIs desenvolvido com **JavaScript** e **k6**, com o objetivo de exercitar endpoints de uma API bancária e observar o seu comportamento durante a execução de cargas controladas.

O projeto permite organizar scripts de teste, reutilizar dados e funções auxiliares e configurar o endereço da API por meio da variável de ambiente `BASE_URL`, evitando a necessidade de fixar o endereço do serviço diretamente nos scripts.

> **Nota:** confirme que a API de destino está disponível e que tem autorização para executar testes de carga sobre ela. Comece com cargas reduzidas e aumente-as gradualmente.

## 1. Introdução

Os testes de performance ajudam a avaliar como uma API se comporta perante diferentes níveis de utilização. Com o k6, é possível simular utilizadores virtuais, executar requisições HTTP e analisar métricas como duração das requisições, taxa de requisições e falhas.

Este repositório reúne scripts e recursos de apoio para a execução de testes de performance da API bancária. A variável `BASE_URL` define a URL base do serviço testado e é fornecida no momento da execução.

## 2. Tecnologias utilizadas

- **JavaScript:** linguagem utilizada para escrever os scripts de teste.
- **[Grafana k6](https://grafana.com/docs/k6/latest/):** ferramenta de código aberto para testes de carga e performance.
- **[Git](https://git-scm.com/) e [GitHub](https://github.com/):** controlo de versões e alojamento do repositório.
- **PowerShell (Windows):** exemplos de comandos para configurar variáveis de ambiente e executar os testes.

## 3. Estrutura do repositório

A estrutura de diretórios identificada no repositório é:

```text
banco-api-performance/
├── config/   # Arquivo de configuração de variáveis de ambiente
├── fixtures/ # Dados de entrada para os testes (ex: usuários, payloads)
├── helpers/  # Funções utilitárias reutilizáveis para interação com a API
├── tests/    # Casos de teste organizados por módulo da API
├── utils/    # Funções utilitárias reutilizá
├── .gitignore
└── README.md
```

## 4. Objetivo de cada grupo de arquivos

| Diretório/arquivo | Objetivo |
|---|---|
| `config/` | Reunir configurações partilhadas pelos testes, quando aplicável. |
| `fixtures/` | Guardar dados de teste, exemplos de payloads ou outros dados reutilizáveis. |
| `helpers/` | Centralizar funções auxiliares reutilizadas pelos scripts de teste. |
| `tests/` | Organizar os scripts que executam os cenários de teste de performance. |
| `utils/` | Reunir utilitários genéricos de apoio à execução dos testes. |
| `.gitignore` | Definir ficheiros e diretórios que não devem ser acompanhados pelo Git, como artefactos gerados localmente. |
| `README.md` | Documentar o objetivo do projeto, a estrutura e os procedimentos de instalação e execução. |

A descrição acima indica a finalidade habitual de cada pasta a partir da sua organização. À medida que o projeto evoluir, atualize esta secção para refletir os ficheiros e as responsabilidades concretas de cada diretório.

## 5. Instalação e execução do projeto

### Pré-requisitos

- Git instalado.
- Grafana k6 instalado e disponível no terminal através do comando `k6`.
- A API de destino acessível a partir do ambiente onde os testes serão executados.

Para instalar o k6, consulte o [guia oficial de instalação](https://grafana.com/docs/k6/latest/set-up/install-k6/).

No Windows, caso utilize o Winget, pode instalar com:

```powershell
winget install k6 --source winget
```

Verifique a instalação:

```powershell
k6 version
```

### Clonar o repositório

```powershell
git clone https://github.com/cintiadiasl/banco-api-performance.git
cd banco-api-performance
```

### Configurar `BASE_URL` e executar um teste

A variável `BASE_URL` deve apontar para a URL base da API que será testada. Nos scripts k6, o valor pode ser lido através de `__ENV.BASE_URL`.

Exemplo de execução no PowerShell:

```powershell
k6 run -e BASE_URL="https://url-da-api" .\tests\nome-do-teste.js
```

Substitua `https://url-da-api` pela URL real da API e `.\tests\nome-do-teste.js` pelo caminho de um script existente no repositório.

Se o script utilizar `__ENV.BASE_URL`, poderá construir os endpoints a partir dessa variável, por exemplo:

```javascript
import http from 'k6/http';

const BASE_URL = __ENV.BASE_URL;

export default function () {
  http.get(`${BASE_URL}/endpoint`);
}
```

O exemplo acima é ilustrativo: ajuste `/endpoint` para uma rota existente na API e mantenha o script alinhado com os cenários e dados do projeto.

### Acompanhar a execução em tempo real e exportar o relatório HTML

O k6 inclui um dashboard web que permite acompanhar métricas enquanto o teste está em execução. É possível também exportar automaticamente um relatório HTML quando o teste terminar. No PowerShell, configure as variáveis de ambiente antes de executar o teste:

```powershell
$env:K6_WEB_DASHBOARD="true"
$env:K6_WEB_DASHBOARD_EXPORT="html-report.html"

k6 run -e BASE_URL="https://url-da-api" .\tests\nome-do-teste.js
```

Durante a execução, abra o dashboard no navegador:

[http://127.0.0.1:5665](http://127.0.0.1:5665)

No final, o ficheiro `html-report.html` será exportado para o diretório de trabalho atual. Substitua o endereço da API e o caminho do script pelos valores corretos do seu ambiente.

Para abrir automaticamente o dashboard no navegador, também pode definir:

```powershell
$env:K6_WEB_DASHBOARD_OPEN="true"
```

Para evitar que estas opções continuem definidas no terminal depois do teste, remova-as com:

```powershell
Remove-Item Env:K6_WEB_DASHBOARD
Remove-Item Env:K6_WEB_DASHBOARD_EXPORT
Remove-Item Env:K6_WEB_DASHBOARD_OPEN -ErrorAction SilentlyContinue
```

**Observação sobre o relatório:** para que o HTML apresente gráficos, a duração do teste deve ser superior a três vezes o período de agregação configurado em `K6_WEB_DASHBOARD_PERIOD`. O período padrão é de 10 segundos. Consulte a [documentação oficial do dashboard web do k6](https://grafana.com/docs/k6/latest/results-output/web-dashboard/) para conhecer as opções disponíveis.

### Exportar também um resumo JSON (opcional)

Se precisar de um resumo agregado em JSON para análise posterior, pode usar a variável de ambiente `K6_SUMMARY_EXPORT`:

```powershell
$env:K6_SUMMARY_EXPORT="summary.json"

k6 run -e BASE_URL="https://url-da-api" .\tests\nome-do-teste.js
```

O ficheiro `summary.json` será criado no diretório de trabalho atual. Este resumo é diferente do relatório HTML do dashboard. Consulte a [documentação das opções do k6](https://grafana.com/docs/k6/latest/using-k6/k6-options/reference/) para mais detalhes.

---

## Referências

- [Documentação oficial do Grafana k6](https://grafana.com/docs/k6/latest/)
- [Instalação do k6](https://grafana.com/docs/k6/latest/set-up/install-k6/)
- [Dashboard web e exportação HTML](https://grafana.com/docs/k6/latest/results-output/web-dashboard/)
- [Repositório no GitHub](https://github.com/cintiadiasl/banco-api-performance)
