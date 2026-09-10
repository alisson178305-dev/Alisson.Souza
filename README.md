# 📘 Ecossistema de Admissão Digital, Automação de DP e POP Operacional

![Status](https://img.shields.io/badge/Status-Conclu%C3%ADdo-brightgreen)
![Google Apps Script](https://img.shields.io/badge/Google%20Apps%20Script-4285F4?style=flat&logo=google&logoColor=white)
![eSocial](https://img.shields.io/badge/eSocial-S--1.2-blue)
![LGPD Compliance](https://img.shields.io/badge/LGPD-Conforme-success)

## 📌 Visão Geral do Projeto

Este projeto consiste em uma solução **Low-Code de Admissão Digital e Automação de Departamento Pessoal**, desenvolvida para otimizar o fluxo de pré-admissão, coleta de documentos e qualificação de colaboradores. 

A solução elimina processos manuais em papel, assegura a integridade dos dados, reduz o tempo de atendimento do RH e garante total conformidade com a **CLT**, as exigências do **eSocial (Versão S-1.2)**, as normas de saúde ocupacional (**NR-07 / ASO**) e a **Lei Geral de Proteção de Dados (LGPD)**.

> **Aplicação Prática:** Projeto estruturado para aplicação em iniciativas operacionais, projetos de voluntariado e gestão de DP, servindo como modelo de otimização de processos e transformação digital de Recursos Humanos.

---

## 📑 1. Procedimento Operacional Padrão (POP) — Admissão e Coleta

### 1.1. Prazos Legais e Matriz de Compliance

| Procedimento Legal | Prazo Máximo Legal | Base Legal / Normativa | Risco Operacional / Penalidade |
| :--- | :--- | :--- | :--- |
| **Exame Admissional (ASO)** | **Anterior** ao início das atividades | Art. 168 CLT / NR-07 | Interdição fiscal, infração sanitária e presunção de nexo causal. |
| **Admissão eSocial (S-2200)** | Até as 23:59h do **dia anterior** ao início | Portaria MTP nº 671/2021 | Multa do Art. 47 da CLT (R$ 800 a R$ 3.000 por empregado). |
| **Admissão Preliminar (S-2190)** | Até as 23:59h do **dia anterior** ao início | Portaria MTP nº 671/2021 | Registro de contingência para evitar a multa do Art. 47 da CLT. |
| **CTPS Digital** | Até 5 dias úteis pós-início | Art. 29 da CLT | Autuação fiscal por manter empregado sem registro formal. |

---

### 1.2. Checklist Documental e Minimização de Dados (LGPD)

* **Documentos Obrigatórios:** RG/CNH, CPF, Comprovante de Residência recente (máximo 90 dias) e ASO Médico Apto.
* **Política de Privacidade (LGPD):** Em conformidade com o Princípio da Minimização de Dados e com o Art. 373-A da CLT, não são solicitados documentos discriminatórios ou desnecessários (como certidões negativas de débitos/SPC, extratos bancários sem finalidade ou testes discriminatórios).

---

### 1.3. Matriz de Gestão de Riscos ("O que fazer quando dá errado?")
/**
 * ECOSSISTEMA COMPLETO DE ADMISSÃO DIGITAL E AUTOMAÇÃO DE DP
 * Automação de Pré-Admissão, Governança no Drive e Layout eSocial (S-2200)
 */

// ===========================================================================
// CONFIGURAÇÕES DO USUÁRIO
// ===========================================================================
var ID_PASTA_RAIZ_DRIVE = "COLE_AQUI_O_ID_DA_SUA_PASTA_DO_DRIVE"; 
var LINK_GOOGLE_FORMS   = "https://docs.google.com/forms/d/e/1FAIpQLSemBgne0oZzLcniX9RWvJgu9shZzbZ03kv1n-LtTKxvtEbNzA/viewform";

/**
 * 1. Cria o menu personalizado na barra superior do Google Sheets ao abrir a planilha
 */
function onOpen() {
  var ui = SpreadsheetApp.getUi();
  ui.createMenu('🚀 Automação DP')
      .addItem('1. Disparar Convocação por E-mail', 'dispararEmailColeta')
      .addItem('2. Gerar XML do eSocial (S-2200)', 'gerarXML_S2200')
      .addToUi();
}

/**
 * 2. Envia e-mail de convocação ao candidato contendo o link direto do formulário
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
    var nome   = sheet.getRange(i, 1).getValue(); // Coluna A (Nome)
    var email  = sheet.getRange(i, 2).getValue(); // Coluna B (E-mail)
    var status = sheet.getRange(i, 5).getValue(); // Coluna E (Status Disparo)

    if (status === "Pendente Disparo" && email !== "") {
      var assunto = "Processo de Admissão - Coleta de Documentos: " + nome;
      var corpo = "Olá, " + nome + "!\n\n" +
                  "Seja bem-vindo(a)! Para dar continuidade ao seu registro profissional, " +
                  "por favor acesse o link abaixo e envie seus dados e documentos:\n\n" +
                  LINK_GOOGLE_FORMS + "\n\n" +
                  "Prazo máximo de preenchimento: 48 horas.\n\n" +
                  "Atenciosamente,\nDepartamento Pessoal";

      MailApp.sendEmail(email, assunto, corpo);
      sheet.getRange(i, 5).setValue("Link Enviado");
      disparos++;
    }
  }
  SpreadsheetApp.getUi().alert(disparos + " e-mail(s) enviado(s) com sucesso!");
}

/**
 * 3. Gatilho Automático (On Form Submit):
 * Cria a pasta individual no Drive (Nome - CPF), move arquivos e grava a URL na Coluna F da planilha.
 */
function aoSubmeterFormulario(e) {
  var aba = SpreadsheetApp.getActiveSpreadsheet().getSheetByName("01_Coleta_Docs");
  var linha = e ? e.range.getRow() : aba.getLastRow();
  
  var nomeCandidato = aba.getRange(linha, 2).getValue(); // Coluna B (Nome)
  var cpfCandidato  = aba.getRange(linha, 3).getValue(); // Coluna C (CPF)
  
  if (!nomeCandidato) return;
  
  // Criar Pasta no Google Drive com o padrão 'Nome - CPF'
  var pastaRaiz = DriveApp.getFolderById(ID_PASTA_RAIZ_DRIVE);
  var nomeNovaPasta = nomeCandidato + (cpfCandidato ? " - " + cpfCandidato : "");
  var novaPasta = pastaRaiz.createFolder(nomeNovaPasta);
  
  // Mover anexos (Colunas D e E)
  var colunasComArquivos = [4, 5]; 
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
  
  // Gravar a URL da pasta criada na Coluna F
  var linkPasta = novaPasta.getUrl();
  aba.getRange(linha, 6).setValue(linkPasta);
}

/**
 * 4. Gera a estrutura do layout XML para o evento S-2200 do eSocial
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
  SpreadsheetApp.getUi().alert("XML S-2200 gerado no Log do Editor!\nConsulte: Execuções > Logs no Apps Script");
}

/**
 * Função Auxiliar: Extrai o ID do arquivo a partir de uma URL do Google Drive
 */
function extrairIdDoDrive(url) {
  var match = url.match(/[-\w]{25,}/);
  return match ? match[0] : null;
}
