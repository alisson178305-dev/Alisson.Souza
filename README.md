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
