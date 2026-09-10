
# 🚀 Projeto de Estruturação e Automação de DP & DHO

![Status](https://img.shields.io/badge/Status-Em_Desenvolvimento-blue)
![Iniciativa](https://img.shields.io/badge/Iniciativa-Trabalho_Volunt%C3%A1rio-orange)
![Google Workspace](https://img.shields.io/badge/Stack-Google_Workspace_--_Apps_Script-4285F4?logo=google)
![Compliance](https://img.shields.io/badge/Compliance-CLT_--_eSocial_--_LGPD-green)

---

## 📌 Visão Geral do Projeto

Este projeto faz parte de uma iniciativa de **trabalho voluntário** focada na estruturação, padronização e automação das áreas de **Departamento Pessoal (DP)** e **Desenvolvimento Humano e Organizacional (DHO)**. 

O objetivo principal é transformar processos manuais e burocráticos em um ecossistema digital fluido, eficiente e seguro, utilizando **Inteligência Artificial (IA)** para o mapeamento de processos e **ferramentas nativas do Google Workspace (Google Sheets, Forms, Docs, Drive e Apps Script)** para automação de ponta a ponta sem custos de licença.

---

## 🗺️ Mapa do Fluxo do Processo (Arquitetura 360°)

┌────────────────────────────────────────────────────────────────────────────────────────┐
│                               1. ENTRADA DE DADOS (RH)                                 │
│  Lançamento dos dados do aprovado na planilha '00_PreCadastro' (Status: Pendente)      │
└───────────────────────────────────────────┬────────────────────────────────────────────┘
│
▼
┌────────────────────────────────────────────────────────────────────────────────────────┐
│                          2. DISPARO AUTOMÁTICO DE CONVOCAÇÃO                           │
│  Acionamento via botão no Google Sheets -> Disparo de e-mail customizado com link     │
└───────────────────────────────────────────┬────────────────────────────────────────────┘
│
▼
┌────────────────────────────────────────────────────────────────────────────────────────┐
│                         3. COLETA DIGITAL DE DOCUMENTOS (CANDIDATO)                    │
│  Preenchimento do Google Forms + Upload de Documentos Pessoais, Bancários e Dependentes│
└───────────────────────────────────────────┬────────────────────────────────────────────┘
│
▼
┌────────────────────────────────────────────────────────────────────────────────────────┐
│                        4. GOVERNANÇA DE ARQUIVOS (APPS SCRIPT + DRIVE)                 │
│  Gatilho 'onFormSubmit' -> Criação da pasta 'Nome - CPF' no Drive + Mover anexos       │
└───────────────────────────────────────────┬────────────────────────────────────────────┘
│
▼
┌────────────────────────────────────────────────────────────────────────────────────────┐
│                         5. AUDITORIA, ASO E QUALIFICAÇÃO ESOCIAL                       │
│  Validação Cadastral + Encaminhamento p/ Exame Admissional (NR-07) + Transmissão S-2200│
└────────────────────────────────────────────────────────────────────────────────────────┘


---

## 📋 1. Procedimento Operacional Padrão (POP) — Admissão e Registro

### 1.1. Checklist Documental Exigido

Para garantir a conformidade trabalhista, tributária e bancária, o formulário de coleta digital exige os seguintes documentos:

* **Documentos de Identificação:** RG ou CNH atualizada e CPF.
* **Registros Trabalhistas:** Número do NIT / PIS / PASEP e CTPS Digital.
* **Comprovação de Residência:** Comprovante de endereço emitido nos últimos 90 dias.
* **Dados Bancários:** Comprovante de titularidade de conta bancária para pagamento de salário.
* **Documentação de Dependentes (Se houver filhos/dependentes):**
  * Certidão de Nascimento dos dependentes.
  * Caderneta de Vacinação (para dependentes menores de 6 anos - Salário-Família).
  * Comprovante de Frequência Escolar (para dependentes a partir de 7 anos).
* **Saúde Ocupacional:** Atestado de Saúde Ocupacional (ASO) Admissional (Apto).

---

### 1.2. Prazos Legais e Matriz de Compliance

| Etapa | Prazo Limite Legal | Base Legal / Normativa | Risco em Caso de Descumprimento |
| :--- | :--- | :--- | :--- |
| **Exame Médico (ASO)** | **Anterior** ao início das atividades | Art. 168 CLT / NR-07 | Interdição fiscal e nulidade da contratação. |
| **Qualificação Cadastral** | Antes de emitir o contrato | Portal eSocial | Rejeição da admissão no eSocial. |
| **eSocial (S-2200 / S-2190)** | Até as 23:59h do **dia anterior** ao início | Portaria MTP nº 671/2021 | Multa do Art. 47 da CLT (R$ 800 a R$ 3.000/empregado). |
| **Registro CTPS Digital** | Até 5 dias úteis pós-início | Art. 29 da CLT | Autuação fiscal do Ministério do Trabalho. |

---

### 1.3. Matriz de Tratamento de Exceções ("O que fazer quando dá errado?")

* **ASO com Resultado "Inapto" ou "Pendente":** Sustar a admissão imediatamente. O candidato não pode assumir o posto de trabalho sem laudo médico definitivo de aptidão.
* **Divergência Cadastral no eSocial (CPF/Nome/Data de Nascimento):** Bloquear o envio do evento S-2200 e orientar o colaborador a regularizar o cadastro junto à Receita Federal ou App Meu INSS.
* **Atraso na Entrega de Documentos:** Transmitir o evento **S-2190 (Admissão Preliminar)** no eSocial até o dia anterior ao início para evitar multas, concedendo prazo de 24 horas para regularização do envio completo.

---

## 🛠️ 2. Arquitetura da Solução Tecnológica

A solução utiliza um ecossistema integrado em **Google Workspace** automatizado via **Google Apps Script**:

* **Google Sheets:** Funciona como banco de dados e painel de controle operacional do RH.
* **Google Forms:** Interface amigável e acessível via celular/computador para o candidato enviar os dados.
* **Google Drive:** Repositório seguro com permissões restritas e organização automática de pastas por colaborador.
* **Google Docs:** Geração de minutas padronizadas de contratos de trabalho e termos operacionais.
* **Google Apps Script:** Código responsável pelo disparo de e-mails, criação de diretórios no Drive e geração da estrutura XML do eSocial.

---

## 💻 3. Código-Fonte da Automação (`codigo_automacao.gs`)

```javascript
/**
 * ECOSSISTEMA DE ADMISSÃO DIGITAL E AUTOMAÇÃO DE DP & DHO
 * Módulo: Pré-Admissão, Coleta de Documentos e eSocial
 */

// CONFIGURAÇÕES GLOBAIS
var ID_PASTA_RAIZ_DRIVE = "SEU_ID_DA_PASTA_DO_DRIVE_AQUI"; 
var LINK_GOOGLE_FORMS   = "SEU_LINK_DO_GOOGLE_FORMS_AQUI";

/**
 * 1. Cria menu personalizado no Google Sheets ao abrir a planilha
 */
function onOpen() {
  var ui = SpreadsheetApp.getUi();
  ui.createMenu('🚀 Automação DP & DHO')
      .addItem('1. Disparar Convite de Admissão por E-mail', 'dispararEmailColeta')
      .addItem('2. Gerar XML do eSocial (S-2200)', 'gerarXML_S2200')
      .addToUi();
}

/**
 * 2. Dispara e-mail de convocação com o link direto do formulário
 */
function dispararEmailColeta() {
  var sheet = SpreadsheetApp.getActiveSpreadsheet().getSheetByName("00_PreCadastro");
  if (!sheet) {
    SpreadsheetApp.getUi().alert("Erro: Aba '00_PreCadastro' não encontrada!");
    return;
  }
  
  var lastRow = sheet.getLastRow();
  var disparos = 0;
  
  for (var i = 2; i <= lastRow; i++) {
    var nome   = sheet.getRange(i, 1).getValue(); // Coluna A
    var email  = sheet.getRange(i, 2).getValue(); // Coluna B
    var status = sheet.getRange(i, 5).getValue(); // Coluna E

    if (status === "Pendente Disparo" && email !== "") {
      var assunto = "Processo de Admissão - Coleta de Documentos: " + nome;
      var corpo = "Olá, " + nome + "!\n\n" +
                  "Seja bem-vindo(a) à nossa equipe! Para darmos início ao seu processo de registro formal, " +
                  "por favor acesse o link abaixo e envie seus dados e documentos obrigatórios:\n\n" +
                  LINK_GOOGLE_FORMS + "\n\n" +
                  "DOCUMENTOS NECESSÁRIOS NO FORMULÁRIO:\n" +
                  "- RG/CNH e CPF\n" +
                  "- Comprovante de Residência recente\n" +
                  "- Número do NIT/PIS e Comprovante Bancário\n" +
                  "- Documentos de Dependentes (se houver filhos)\n\n" +
                  "Prazo máximo de preenchimento: 48 horas.\n\n" +
                  "Atenciosamente,\nEquipe de DP & DHO";

      MailApp.sendEmail(email, assunto, corpo);
      sheet.getRange(i, 5).setValue("Link Enviado");
      disparos++;
    }
  }
  SpreadsheetApp.getUi().alert(disparos + " e-mail(s) enviado(s) com sucesso!");
}

/**
 * 3. Gatilho Automático (On Form Submit): Cria pasta Nome-CPF no Drive e move anexos
 */
function aoSubmeterFormulario(e) {
  var aba = SpreadsheetApp.getActiveSpreadsheet().getSheetByName("01_Coleta_Docs");
  var linha = e ? e.range.getRow() : aba.getLastRow();
  
  var nomeCandidato = aba.getRange(linha, 2).getValue(); 
  var cpfCandidato  = aba.getRange(linha, 3).getValue(); 
  
  if (!nomeCandidato) return;
  
  // Criar Pasta no Google Drive com o padrão 'Nome - CPF'
  var pastaRaiz = DriveApp.getFolderById(ID_PASTA_RAIZ_DRIVE);
  var nomeNovaPasta = nomeCandidato + (cpfCandidato ? " - " + cpfCandidato : "");
  var novaPasta = pastaRaiz.createFolder(nomeNovaPasta);
  
  // Mover anexos (Colunas D, E, F e subsequentes com uploads)
  var colunasComArquivos = [4, 5, 6, 7]; 
  colunasComArquivos.forEach(function(coluna) {
    var urlArquivo = aba.getRange(linha, coluna).getValue();
    if (urlArquivo && urlArquivo.toString().indexOf("http") !== -1) {
      var fileId = extrairIdDoDrive(urlArquivo);
      if (fileId) {
        var arquivo = DriveApp.getFileById(fileId);
        arquivo.moveTo(novaPasta);
      }
    }
  });
  
  // Registrar link da pasta na Coluna H da planilha
  var linkPasta = novaPasta.getUrl();
  aba.getRange(linha, 8).setValue(linkPasta);
}

/**
 * 4. Gera o leiaute XML do evento S-2200 do eSocial
 */
function gerarXML_S2200() {
  var aba = SpreadsheetApp.getActiveSpreadsheet().getSheetByName("00_PreCadastro");
  var nome = aba.getRange("A2").getValue();
  var cargo = aba.getRange("D2").getValue();
  
  var abaDocs = SpreadsheetApp.getActiveSpreadsheet().getSheetByName("01_Coleta_Docs");
  var cpf = abaDocs ? abaDocs.getRange("C2").getValue() : "000.000.000-00";
  
  var xml = '<?xml version="1.0" encoding="UTF-8"?>\n' +
            '<eSocial xmlns="http://www.esocial.gov.br/schema/evt/evtAdmissao/v_S_01_02_00">\n' +
            '  <evtAdmissao id="ID1123456780000002026090809300000001">\n' +
            '    <ideEmpregador>\n' +
            '      <tpInsc>1</tpInsc>\n' +
            '      <nrInsc>12345678000199</nrInsc>\n' +
            '    </ideEmpregador>\n' +
            '    <trabalhador>\n' +
            '      <cpfTrab>' + cpf + '</cpfTrab>\n' +
            '      <nmTrab>' + nome + '</nmTrab>\n' +
            '    </trabalhador>\n' +
            '    <vinculo>\n' +
            '      <infoRegimeTrab>\n' +
            '        <infoCeletista>\n' +
            '          <dtAdm>2026-09-15</dtAdm>\n' +
            '          <tpAdmissao>1</tpAdmissao>\n' +
            '        </infoCeletista>\n' +
            '      </infoRegimeTrab>\n' +
            '      <infoContrato>\n' +
            '        <nmCargo>' + cargo + '</nmCargo>\n' +
            '      </infoContrato>\n' +
            '    </vinculo>\n' +
            '  </evtAdmissao>\n' +
            '</eSocial>';
            
  Logger.log(xml);
  SpreadsheetApp.getUi().alert("XML S-2200 gerado com sucesso nos Logs do Apps Script!");
}

function extrairIdDoDrive(url) {
  var match = url.match(/[-\w]{25,}/);
  return match ? match[0] : null;
}
📈 4. Roadmap e Próximas Etapas
[x] Módulo 1: Mapeamento de Admissão, Coleta Digital e Governança no Drive.

[ ] Módulo 2: Gestão e Concessão de Benefícios (VT, VR/VA, Plano de Saúde).

[ ] Módulo 3: Estruturação da Folha de Pagamento e Encargos (INSS, IRRF, FGTS).

[ ] Módulo 4 (DHO): Programa de Onboarding e Avaliação de Desempenho de Experiência.

📊 Estatísticas do Repositório
👥 Contribuição e Licença
Projeto desenvolvido como iniciativa voluntária de estruturação organizacional e automação de processos de Recursos Humanos.
