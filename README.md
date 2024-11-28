//Trigger Class

trigger ContactTrigger on Contact (after Insert , after Update , After Delete) {
    if(Trigger.isInsert){
        if(Trigger.isAfter){
            ContactHandler.OnlyOnePrimaryContactOfSameAccount(Trigger.new);
        }
    }
    if(Trigger.isUpdate){
        if(Trigger.isAfter){
            ContactHandler.OnlyOnePrimaryContactOfSameAccount(Trigger.new);
        }
    }
    if(Trigger.isDelete){
        if(Trigger.isAfter){
            ContactHandler.OnlyOnePrimaryContactOfSameAccount(Trigger.old);
        }
    }
}

//Handler Class 

public class ContactHandler {
   public static void OnlyOnePrimaryContactOfSameAccount(List<Contact> conList){
        Set<Id> accIds = new Set<Id>();
        for(Contact con : conList){
            if(con.PrimaryContact__c == true){
                accIds.add(con.AccountId);
            }
        }
        Map<Id,Account> accMap = new Map<Id,Account>();
        for(Account acc : [SELECT Id , (SELECT Id , PrimaryContact__c FROM Contacts WHERE PrimaryContact__c=true) FROM Account WHERE Id IN : accIds]){
            accMap.put(acc.Id , acc);
        }
        for(Contact con : conList){
            if(con.AccountId!=NULL && con.PrimaryContact__c == true && accMap.get(con.AccountId).contacts.size()>1){
                con.PrimaryContact__c.AddError('You cant have primary account more than one');
            }
        }
    }
}
