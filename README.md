# Projeto-Endereco
Crie um objeto chamado endereço, este objeto deverá conter campos, a cep, logradouro, complemento, bairro, localidade uf. O registro deverá ser criado informando apenas o CEP os demais campos, logradouro, complemento, bairro, localidade, uf deverão ser inseridos automaticamente após clicar em salvar vi CEP

public class EnderecoAtualizado2 {
    @future (callout=true)
    public static void EnderecoBeforeAtualizado2(List<Id> enderecoIds) {
        String site = 'viacep.com.br/ws/';
        String compl = '/json/'
        if(enderecoIds.size() > 0){
        for (Endereco__c endereco : [select Id, CEP__c, Logradouro__c, Complemento__c, Bairro__c, Localidade__c, UF__c from Endereco__c where Id IN :enderecoIds]) {
            if (endereco.CEP__c != null && endereco.CEP__c > 0) {
                String urlCompleta = site + endereco.CEP__c + compl;
                HttpRequest request = new HttpRequest();
                request.setEndpoint(urlCompleta);
                request.setMethod('GET');  
                HttpResponse response = new Http().send(request);
                System.debug('Status code is'+ response.getStatusCode());
                if (response.getStatusCode() == 200) {
                    // A resposta foi bem-sucedida, analise os dados do JSON retornado
                    Map<String, Object> responseData = (Map<String, Object>) JSON.deserializeUntyped(response.getBody());
                    // Atualize os campos do objeto Endereco__c com os valores retornados
                    Endereco.Logradouro__c = (String) responseData.get('logradouro');
                    endereco.Complemento__c = (String) responseData.get('complemento');
                    endereco.Bairro__c = (String) responseData.get('bairro');
                    endereco.Localidade__c = (String) responseData.get('localidade');
                    endereco.UF__c = (String) responseData.get('uf');
                    System.debug('Sucesso');
                    update endereco;
                } else {
                    // A chamada à API falhou, lide com isso conforme necessário
                    endereco.addError('Erro ao consultar a API Via CEP. Por favor, verifique o CEP e tente novamente.');
                }
            } else {
                // CEP em branco, não há necessidade de consultar a API
                endereco.addError('O campo CEP não pode estar em branco.');
            }
        }




   trigger EnderecoTrigger2 on Endereco__c (before insert, before update) {
	 List<Id> enderecoIds = new List<Id>();

    for (Endereco__c endereco : Trigger.new) {
        enderecoIds.add(endereco.Id);
    }

    if (!enderecoIds.isEmpty()) {
        EnderecoAtualizado2.EnderecoBeforeAtualizado2(enderecoIds);
    }
}

        }}
}
