# Bank Account System 

---

## Step 1: Create the Custom Object (Database Table)

First, we need to create a custom object to store the customer records.

1. Log in to your Salesforce Developer Edition.
2. Click the **Gear Icon** (top right) and select **Setup**.
3. Navigate to **Object Manager** (top navigation bar) -> **Create** -> **Custom Object**.
4. Fill in the following details:
   * **Label:** Customer
   * **Plural Label:** Customers
   * **Object Name:** Customer
   * **Record Name:** Emp Name
   * **Data Type:** Text
5. Scroll down and click **Save**.

---

## Step 2: Create Custom Fields (Table Columns)

Now, add the required fields to the `Customer` object.

1. In the Object Manager for your new `Customer` object, click **Fields & Relationships** on the left sidebar.
2. Click the **New** button to create the following fields one by one:

| Data Type | Field Label | Length | Field Name (API Name) |
| :--- | :--- | :--- | :--- |
| **Text** | Emp ID | 10 | `Emp_ID__c` |
| **Email** | Email | - | `Email__c` |
| **Date** | Birth Date | - | `Birth_Date__c` |
| **Text** | Department | 50 | `Department__c` |

*(Note: The API Name gets generated automatically when you type the Field Label. Make sure they match the table above exactly, as the Apex code relies on these exact names).*

---

## Step 3: Create the Apex Class (Backend Menu Logic)

This class acts as our menu-driven console program, handling Insert, Read, Update, and Delete (CRUD) operations.

1. Click the **Gear Icon** -> **Developer Console**.
2. Go to **File** -> **New** -> **Apex Class**.
3. Name the class `BankAccountSystem` and click **OK**.
4. Paste the following code and save (`Ctrl + S` or `Cmd + S`):

```apex
public class BankAccountSystem {

    // Main Menu Method
    public static void menu(Integer choice, String empId, String empName, String email, Date dob, String dept) {
        switch on choice {
            when 1 { insertCustomer(empId, empName, email, dob, dept); }
            when 2 { viewCustomers(); }
            when 3 { updateCustomerEmail(empId, email); }
            when 4 { deleteCustomer(empId); }
            when else { System.debug('Invalid Choice! Menu: 1.Insert | 2.View | 3.Update Email | 4.Delete'); }
        }
    }

    // 1. Insert Record
    public static void insertCustomer(String empId, String empName, String email, Date dob, String dept) {
        Customer__c newCust = new Customer__c(
            Name = empName, 
            Emp_ID__c = empId,
            Email__c = email,
            Birth_Date__c = dob,
            Department__c = dept
        );
        insert newCust;
        System.debug('SUCCESS: Customer Record Inserted for ' + empName);
    }

    // 2. View Records
    public static void viewCustomers() {
        List<Customer__c> custList = [SELECT Emp_ID__c, Name, Email__c, Birth_Date__c, Department__c FROM Customer__c];
        System.debug('--- CUSTOMER RECORDS ---');
        for(Customer__c c : custList) {
            System.debug('ID: ' + c.Emp_ID__c + ' | Name: ' + c.Name + ' | Email: ' + c.Email__c + ' | Dept: ' + c.Department__c);
        }
    }

    // 3. Update Record
    public static void updateCustomerEmail(String empId, String newEmail) {
        List<Customer__c> custList = [SELECT Id, Email__c FROM Customer__c WHERE Emp_ID__c = :empId LIMIT 1];
        if(!custList.isEmpty()) {
            custList[0].Email__c = newEmail;
            update custList[0];
            System.debug('SUCCESS: Customer Email Updated to: ' + newEmail);
        } else {
            System.debug('ERROR: Customer with Emp ID ' + empId + ' not found.');
        }
    }

    // 4. Delete Record
    public static void deleteCustomer(String empId) {
        List<Customer__c> custList = [SELECT Id FROM Customer__c WHERE Emp_ID__c = :empId LIMIT 1];
        if(!custList.isEmpty()) {
            delete custList;
            System.debug('SUCCESS: Customer Record Deleted.');
        } else {
            System.debug('ERROR: Customer with Emp ID ' + empId + ' not found.');
        }
    }
}
```

## Step 4 : Execute & Test the Code (The "Console" Output)

Since Salesforce doesn't have a traditional terminal, we use the Execute Anonymous Window to simulate our menu choices and pass data.
1) In the Developer Console, press Ctrl + E (or go to Debug -> Open Execute Anonymous Window).
2) Check the Open Log box at the bottom.
3) Paste one of the commands below, click Execute, and then check the Debug Only filter in the log to see your output.

1. Insert Records
```
BankAccountSystem.menu(1, 'E001', 'Alice Johnson', 'alice@example.com', Date.newInstance(1999, 4, 12), 'Computer Science');
BankAccountSystem.menu(1, 'E002', 'Bob Smith', 'bob@example.com', Date.newInstance(2001, 8, 25), 'Information Technology');
```

3. View All Records
```
// Pass null for data we don't need
BankAccountSystem.menu(2, null, null, null, null, null);
```

4. Update an Email (using Emp ID)
```
BankAccountSystem.menu(3, 'E001', null, 'alice.new@company.com', null, null); 
```

5. Delete a Record (using Emp ID)
```
BankAccountSystem.menu(4, 'E002', null, null, null, null);
```

## Step 5: (Optional Bonus) Visualforce Page GUI
1. Create the Apex Controller 
If your examiner asks for a User Interface instead of a console log, follow these steps to turn it into a web page.
Developer Console -> File -> New -> Apex Class. Name it CustomerUIController.

```
public class CustomerUIController {
    public Customer__c cust { get; set; }
    public String searchEmpId { get; set; }
    public List<Customer__c> customerList { get; set; }
    
    public CustomerUIController() {
        cust = new Customer__c();
        loadCustomers();
    }
    
    public void loadCustomers() {
        customerList = [SELECT Emp_ID__c, Name, Email__c, Department__c FROM Customer__c ORDER BY CreatedDate DESC];
    }
    
    public PageReference saveCustomer() {
        try {
            insert cust;
            cust = new Customer__c(); // Clear form
            loadCustomers(); // Refresh list
        } catch(Exception e) {
            ApexPages.addMessages(e);
        }
        return null;
    }
    
    public PageReference deleteCustomer() {
        if(String.isNotBlank(searchEmpId)) {
            List<Customer__c> toDelete = [SELECT Id FROM Customer__c WHERE Emp_ID__c = :searchEmpId LIMIT 1];
            if(!toDelete.isEmpty()) {
                delete toDelete;
                loadCustomers();
            }
        }
        return null;
    }
}
```
2. Create the Visualforce Page
Developer Console -> File -> New -> Apex Class. Name it CustomerUIController.
```
<apex:page controller="CustomerUIController" docType="html-5.0">
    <apex:form >
        <apex:pageMessages />
        
        <apex:pageBlock title="Bank Account System">
            <apex:pageBlockSection title="Add New Customer" columns="2">
                <apex:inputField value="{!cust.Emp_ID__c}" required="true"/>
                <apex:inputField value="{!cust.Name}" required="true"/>
                <apex:inputField value="{!cust.Email__c}"/>
                <apex:inputField value="{!cust.Department__c}"/>
                <apex:inputField value="{!cust.Birth_Date__c}"/>
            </apex:pageBlockSection>
            <apex:pageBlockButtons location="bottom">
                <apex:commandButton value="Save Customer" action="{!saveCustomer}"/>
            </apex:pageBlockButtons>
        </apex:pageBlock>
        
        <apex:pageBlock title="Delete Customer">
            <apex:outputLabel value="Enter Emp ID to Delete: " />
            <apex:inputText value="{!searchEmpId}"/>
            <apex:commandButton value="Delete" action="{!deleteCustomer}" style="margin-left:10px;"/>
        </apex:pageBlock>
        
        <apex:pageBlock title="All Customers">
            <apex:pageBlockTable value="{!customerList}" var="c">
                <apex:column value="{!c.Emp_ID__c}" headerValue="Emp ID"/>
                <apex:column value="{!c.Name}" headerValue="Name"/>
                <apex:column value="{!c.Email__c}" headerValue="Email"/>
                <apex:column value="{!c.Department__c}" headerValue="Department"/>
            </apex:pageBlockTable>
        </apex:pageBlock>
    </apex:form>
</apex:page>
```
