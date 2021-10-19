# BuscaCEPController.js

({   /* TOAST MSG NA TELA DO USUARIO */
    doInit : function(component, event, helper) {
        console.log(component.get('v.recordId'));      
        
    },  
    
    /* BOTAO DE CANCELAR */
    cancelar : function (component, event, helper){
        var dismissActionPanel = $A.get("e.force:closeQuickAction");
        dismissActionPanel.fire();  /* close quick action fecha janela*/
    },
    
    /* BOTAO DO ICONE */
    buscarCEP : function (component, event, helper){
        
      /* PRECISAMOS QUE A FUNÇÃO SE COMUNIQUE COM O APEX https://developer.salesforce.com/docs/atlas.en-us.lightning.meta/lightning/controllers_server_actions_call.htm*/
        var action = component.get("c.consultaCep");
        action.setParams({ 
            cepDigitado : component.get("v.CEP") /* PARAMETRO E ATRIBUTO */
        });
        
        action.setCallback(this, function(response) {
            var state = response.getState();
            if (state === "SUCCESS") {               
                console.log(response.getReturnValue());
                component.set("v.retornoViaCEP", response.getReturnValue()); /* DA VALOR AO ATRIBUTO */ 
                var toastEvent = $A.get("e.force:showToast");
                toastEvent.setParams({
                    "title": "Success!", /* ESTE TOAST VAI APARECER QUANDO CEP FOR VALIDO */
                    "message": "consulta realizada com sucesso",   
                    "type":"success" /* ATRIBUTO CHAMADO TYPE */
                });
                toastEvent.fire();
            }else {
                console.log("erro");
            }
        });                
        $A.enqueueAction(action);
    },
    
    /* BOTAO DE SALVAR - INSERIR NO ENDEREÇO DE COBRANÇA */
     salvarEndereco : function (component, event, helper){
      
        var action = component.get("c.atualizaEndereco");
        action.setParams({ 
            idConta : component.get("v.recordId"),
            jsonViaCep : JSON.stringify(component.get("v.retornoViaCEP")) /* CONSEGUE TRANSFORMAR JSON EM UMA STRING NOVAMENTE https://developer.mozilla.org/pt-BR/docs/Web/JavaScript/Reference/Global_Objects/JSON/stringify */
        });
        
        action.setCallback(this, function(response) {
            var state = response.getState();
            if (state === "SUCCESS") {               
                console.log(response.getReturnValue());
                if(response.getReturnValue()){
                    var toastEvent = $A.get("e.force:showToast");
                    toastEvent.setParams({
                        "title": "Success!",
                        "message": "Atualizado com sucesso",   
                        "type":"success"
                    });
                    toastEvent.fire();
                    $A.get('e.force:refreshView').fire();
                    var dismissActionPanel = $A.get("e.force:closeQuickAction");
                    dismissActionPanel.fire();
                    
                }else{
                    console.log("erro");
                }
                
            }else {
                console.log("erro");
            }
        });                
        $A.enqueueAction(action);
    },
})
