# Test
<apex:page title="{!Lead__c.Name} -- {!Lead__c.Last_name__c} -- Student Loan" id="studentLead" standardstylesheets="false" standardController="Lead__c" extensions="StudentLeadController" >
   <apex:includeScript value="{!URLFOR($Resource.student, 'jquery-1.10.2.min.js')}"/>
   <apex:includeScript value="{!URLFOR($Resource.student, 'script.js')}"/>
   <apex:stylesheet value="{!URLFOR($Resource.student, 'style.css')}"/>
      <c:Disposition LeadIdName="{!Lead__c.Id}" />
      <apex:form id="submitform">
      <apex:actionFunction action="{!Save}" name="autoSave" />
      <input type="hidden" id="quoteId" />           
      <script type="text/javascript">

      if ( '{!updated}' == 'true' )
      {
         autoSave();
      }
    
      function setUrl() {
         //document.getElementById('{!$Component.contractUrl}').value = 
         /* this line  of code is commented as per client instruction for more information see this ticket #IN00002125
            &ServerUrl={!$Api.Partner_Server_URL_80} 
          var url = contractUrl+'&SessionId={!$Api.Session_ID}&ServerUrl={!$Api.Partner_Server_URL_80}';
         */
         var url = contractUrl+'&SessionId={!$Api.Session_ID}';
         window.open(url,'name','height=450,width=650');
      }
      </script>
      <div style="display: none;">
         <apex:inputText value="{!urlFromPage}" id="contractUrl" />
      </div>
      <apex:dynamicComponent componentValue="{!leadButtons}" rendered="true" />
      </apex:form>
      <apex:pageMessages id="pageMessage"/>

      <!-- Lead Summary -->
      <apex:pageBlock id="leadSummary">
         <apex:pageBlockSection columns="2">
            <table style="border-collapse: collapse; width: 100%; border: 1px solid #000;">
               <caption style="background: #99CC66; border: 1px solid #000; border-bottom: 0px; font-size: 1.3em;">
                  <strong>Debtor - {!Lead__c.First_Name__c} {!Lead__c.Last_name__c} ({!Lead__c.Name})</strong><br/>
               </caption>
               <tr>
                  <td style="font-weight: bold;">
                     Best Phone
                  </td>
                  <td>
                     {!Lead__c.Best_Phone__c}
                  </td>
                  <td style="font-weight: bold;">
                     Home Phone
                  </td>
                  <td>
                     {!Lead__c.Home_Phone__c}
                  </td>
               </tr>
               <tr>
                  <td style="font-weight: bold;">
                     Cell Phone
                  </td>
                  <td>
                     {!Lead__c.Cell_Phone__c}
                  </td>
                  <td style="font-weight: bold;">
                     Email
                  </td>
                  <td>
                     {!Lead__c.Email_Address__c}
                  </td>
               </tr>
               <tr>
                  <td style="font-weight: bold; vertical-align: top;">
                     Address
                  </td>
                  <td colspan="3">
                     {!Lead__c.Address_Line_1__c}<br/>
                     {!Lead__c.Address_Line_2__c}<br/>
                     {!Lead__c.City__c}, {!Lead__c.State__c} {!Lead__c.Postal_Code__c}
                  </td>
               </tr>
            </table>
            <apex:pageBlockSectionItem id="statusblock">
            <apex:form >
               <apex:outputPanel id="statusblock">
               <div style="float: left;">
               <apex:outputLabel for="leadstatus" style="font-weight: bold;" value="Lead Status: "/>
               <br/>
               <apex:outputField rendered="{!$Setup.SLE_Config__c.Lock_Lead_Status__c == true}" label="Lead Status: " value="{!Lead__c.Lead_Status__c}">
               </apex:outputField>
               <apex:inputField rendered="{!$Setup.SLE_Config__c.Lock_Lead_Status__c == false}" label="Lead Status: " value="{!Lead__c.Lead_Status__c}">
                   <apex:actionSupport action="{!Save}" event="onchange" />
               </apex:inputfield>
               </div>
               <div style="float: left; margin-left: 20px;">
               <apex:outputLabel for="leadowner" style="font-weight: bold; float: left;" value="Lead Owner: "/>
               <br/>
               <apex:inputField label="Lead Owner: " id="leadowner" value="{!Lead__c.OwnerId}" style="float: left;">
                  <apex:actionSupport action="{!Save}" event="onchange" />
               </apex:inputField>
               </div>
               <div style="clear: both;"></div>
               <apex:outputPanel layout="block">
                  <apex:outputLabel for="affiliate" style="font-weight: bold; float: left;" value="Affiliate: "/>
                  <br/>
                  <apex:inputField id="affiliate" value="{!Lead__c.Affiliate__c}">
                     <apex:actionSupport action="{!Save}" event="onchange"/>
                  </apex:inputField>
               </apex:outputPanel>
               </apex:outputPanel>
            </apex:form>
            </apex:pageBlockSectionItem>
         </apex:pageBlockSection>
      </apex:pageBlock>

      <!-- tabs -->
      <apex:form >
         <apex:actionFunction name="updateLastTab" action="{!setLastTab}" rerender="this">
            <apex:param name="tabName" value=""/>
         </apex:actionFunction>
         <apex:actionFunction name="apexDoAction" action="{!doAction}" rerender="refreshme,quoterefresh" status="retrieveStatus" oncomplete="init();">
            <apex:param name="selectedLoans" value="" assignTo="{!selectedLoans}" />
            <apex:param name="selectedAction" value="" assignTo="{!selectedAction}" />
         </apex:actionFunction>
      </apex:form>
      <apex:tabPanel switchType="client" value="{!selectedTab}" id="studentTabs" tabClass="activeTab" inactiveTabClass="inactiveTab">
         <apex:tab label="Lead Detail" name="leadDetail" id="leadDetail" onTabEnter="updateLastTab('leadDetail');" >
            <apex:form >
               <apex:pageBlock id="leaddetailcont">
                  <apex:pageBlockButtons >
                     <apex:commandButton value="Save" action="{!Save}" />
                     <apex:commandButton value="Cancel" action="{!Cancel}" />
                  </apex:pageBlockButtons>
                  <apex:outputPanel id="PersonalInfo">
                     <apex:outputPanel rendered="{!IF(leadDetailTop!=null,true,false)}">
                        <apex:pageBlockSection columns="2" title="{!leadDetailTop.label}">
                           <apex:repeat value="{!leadDetailTop.fields}" var="f">
                              <apex:inputField value="{!lead[f]}" rendered="{!IF($ObjectType.Lead__c.fields[f].calculated == false, true, false)}"/>
                              <apex:pageBlockSectionItem rendered="{!IF($ObjectType.Lead__c.fields[f].calculated == true, true, false)}">
                                 <apex:outputLabel value="{!$ObjectType.Lead__c.fields[f].Label}"/>
                                 <apex:outputText value="{!formulaFields[f]}" rendered="{!formulaFieldHasValue[f]}"/>
                              </apex:pageBlockSectionItem>
                           </apex:repeat>
                        </apex:pageBlockSection>
                     </apex:outputPanel>
                  <apex:pageBlockSection columns="2" title="Contact Details">
                     <apex:inputField styleClass="required" tabOrderHint="101" value="{!Lead__c.First_Name__c}"/>
                     <apex:inputField tabOrderHint="151" value="{!Lead__c.Address_Line_1__c}"/>
                     <apex:inputField tabOrderHint="102" value="{!Lead__c.MI__c}"/>
                     <apex:inputField tabOrderHint="152" value="{!Lead__c.Address_Line_2__c}"/>
                     <apex:inputField tabOrderHint="103" styleClass="required" value="{!Lead__c.Last_name__c}"/>
                     <apex:inputField tabOrderHint="153" value="{!Lead__c.City__c}"/>
                     <apex:inputField tabOrderHint="104" value="{!Lead__c.Email_Address__c}"/>
                     <apex:inputField tabOrderHint="154" value="{!Lead__c.State__c}"/>
                     <apex:inputField tabOrderHint="105" value="{!Lead__c.Best_Phone__c}"/>
                     <apex:inputField tabOrderHint="155" value="{!Lead__c.Postal_Code__c}"/>
                     <apex:inputField tabOrderHint="106" value="{!Lead__c.Home_Phone__c}"/>
                     <apex:inputField tabOrderHint="156" value="{!Lead__c.Marital_Status__c}">
                        <apex:actionSupport action="{!updateMarriage}" event="onchange" rerender="SpouseInfo,PersonalInfo,refreshme"/>
                     </apex:inputField>
                     <apex:inputField tabOrderHint="107" value="{!Lead__c.Cell_Phone__c}"/>
                     <apex:inputField tabOrderHint="157" value="{!Lead__c.Preferred_Language__c}"/>
                     <apex:inputField tabOrderHint="108" value="{!Lead__c.Work_Phone__c}"/>
                     <apex:inputField value="{!Lead__c.Previous_Names__c}"/>
                     <apex:inputField tabOrderHint="109" value="{!Lead__c.Estimated_AGI__c}"/>
                     <apex:inputField tabOrderHint="158" value="{!Lead__c.Household_Size__c}"/>
                     <apex:inputField tabOrderHint="110" value="{!Lead__c.Taxes_Filed__c}" rendered="{!isMarried}">
                        <apex:actionSupport action="{!updateTaxes}" event="onchange" rerender="SpouseInfo"/>
                     </apex:inputField>
                     <apex:inputField value="{!Lead__c.DLST__c}"/>
                     <apex:inputField value="{!Lead__c.DL__c}"/>
                     <apex:inputField value="{!Lead__c.SSN_ENC__c}"/>
                     <apex:inputField value="{!Lead__c.DOB__c}"/>
                     <apex:inputField value="{!Lead__c.FAFSA_PIN_ENC__c}"/>
                  </apex:pageBlockSection>
                  </apex:outputPanel>
                  <apex:outputPanel id="SpouseInfo">
                     <apex:pageBlockSection title="Spouse Details" rendered="{!isMarried}" >
                        <apex:inputField tabOrderHint="160" value="{!Lead__c.CO_First_Name__c}"/>
                        <apex:inputField tabOrderHint="161" value="{!Lead__c.CO_Last_Name__c}"/>
                        <apex:inputField value="{!Lead__c.CO_SSN_ENC__c}"/>
                        <apex:inputField value="{!Lead__c.CO_DOB__c}"/>
                        <apex:inputField tabOrderHint="162" rendered="{!isSeparateTax}" value="{!Lead__c.Estimated_AGI_Spouse__c}"/>
                     </apex:pageBlockSection>
                  </apex:outputPanel>
                  <apex:pageBlockSection title="Source Information">
                     <apex:inputField tabOrderHint="170" value="{!Lead__c.Source__c}"/>
                     <apex:inputField tabOrderHint="171" value="{!Lead__c.Source_ID__c}"/>
                     <apex:inputField tabOrderHint="172" value="{!Lead__c.Campaign_ID__c}"/>
                     <apex:inputField tabOrderHint="173" value="{!Lead__c.SourceIdentifier2__c}"/>
                  </apex:pageBlockSection>
                  <apex:outputPanel rendered="{!IF(leadDetailBottom!=null,true,false)}">
                     <apex:pageBlockSection columns="2" title="{!leadDetailBottom.label}">
                        <apex:repeat value="{!leadDetailBottom.fields}" var="f">
                           <apex:inputField value="{!lead[f]}" rendered="{!IF($ObjectType.Lead__c.fields[f].calculated == false, true, false)}"/>
                           <apex:pageBlockSectionItem rendered="{!IF($ObjectType.Lead__c.fields[f].calculated == true, true, false)}">
                              <apex:outputLabel value="{!$ObjectType.Lead__c.fields[f].Label}"/>
                              <apex:outputText value="{!formulaFields[f]}" rendered="{!formulaFieldHasValue[f]}"/>
                           </apex:pageBlockSectionItem>
                        </apex:repeat>
                     </apex:pageBlockSection>
                  </apex:outputPanel>
               </apex:pageBlock>
            </apex:form>
         </apex:tab>
         <!-- Loan Details Tab -->
         <style type="text/css">
            .sleWarn
            {
               padding-left: 18px;
            }

            .sleClear
            {
            }

            .sleNone
            {
            }

            .sleRed
            {
               background: #FF3;
            }

            .sleEst
            {
               background: #CC3;
            }
            .table1{border:1px solid #ccc; }
              .table1 tr td{padding:5px; border-right:1px solid #ccc;}
            .table1 tr th{padding:5px; border-right:1px solid #ccc;}
            
            .table{border:1px solid #ccc; float:left; width:50%;}
            .table tr td{padding:5px; border-right:1px solid #ccc;}
            .table tr th{padding:5px; border-right:1px solid #ccc;}
         </style>
         <apex:tab label="Loan Detail" name="loanDetails" id="loanDetails" onTabEnter="updateLastTab('loanDetails');">
            <apex:form id="loandetailform">
               <apex:pageBlock id="loandetailcont">
                  <apex:pageBlockButtons id="mbuttons">
                     <apex:commandButton value="Save" action="{!Save}" />
                     <apex:commandButton value="Cancel" action="{!Cancel}" />
                  </apex:pageBlockButtons>
               <apex:outputPanel id="refreshme">
                  <apex:pageBlockSection columns="2" title="Loan Summary" id="loanSummary">
                     <apex:outputField label="Current Loan Balance" value="{!Lead__c.Loan_Amount__c}"/>                  
                     <apex:outputField label="Loan Amount Borrowed" value="{!Lead__c.Loan_Amount_Borrowed__c}"/>
                     <apex:outputField label="Current Monthly Payment" value="{!Lead__c.Monthly_Student_Payment__c}"/>
                     <apex:outputField label="Estimated Monthly Payment" value="{!Lead__c.CALC_Monthly_Student_Payment__c}"/>
                     <apex:outputField label="Current Annual Interest Rate" value="{!Lead__c.Interest_Rate__c}"/>
                     <apex:inputField tabOrderHint="201" value="{!Lead__c.Student_Loan_Type__c}"/>
                     <apex:inputField tabOrderHint="202" value="{!Lead__c.Student_Loan_Status__c}"/>
                     <apex:inputField tabOrderHint="203" value="{!Lead__c.Works_in_Public_Service__c}"/>                                          
                  </apex:pageBlockSection>
                  <apex:pageBlockSection title="Spouse Loan Details" rendered="{!IF(isMarried==true && isSeparateTax==false,true,false)}">
                     <apex:inputField tabOrderHint="204" label="Spouse Current Loan Balance" value="{!Lead__c.Loan_Amount_Spouse__c}"/>
                     <apex:inputField tabOrderHint="205" label="Spouse Loan Amount Borrowed" value="{!Lead__c.Loan_Amount_Borrowed_Spouse__c}"/>                     
                     <apex:inputField tabOrderHint="206" label="Spouse Annual Interest Rate" value="{!Lead__c.Interest_Rate_Spouse__c}"/>
                  </apex:pageBlockSection>
                  <apex:pageBlockSection html-data-id="loan-tb" title="Loans" id="loanBlock" columns="1">
                  <!--<iframe src="https://www.nslds.ed.gov/nslds_SA/SaLogoff.do" style="display:none" />
                  <iframe src="" id="loginNsldsFrameGet" style="display:none" />
                     <script>
                     function getFilesforClient(){
                         
                         var leadMainId = document.getElementById("leadMainId").value;
                         var leadMainIdUrl = "{!$Page.RetrieveLoanStep1}?leadMainId="+leadMainId;
                         document.getElementById("loginNsldsFrameGet").src = leadMainIdUrl;
                        
                     }
                     
                     
                     </script>
                     <input type="hidden" id="leadMainId" name="leadMainId" value="{!Lead__c.id}"/>
                     <apex:outputPanel rendered="{!IF(Lead__c.SSN_ENC__c != '',true,false)}">
                     <a href="javascript:void(0)" onclick="getFilesforClient()"  >Get NSLDS Loan File</a>
                     </apex:outputPanel>-->
                     <apex:outputLink value="{!URLFOR($Action.Lead__c.NSLDS_Student_Access)}" target="_blank">NSLDS Student Access</apex:outputLink>
                     <apex:outputLink value="{!URLFOR($Action.Lead__c.FAFSA_PIN_Access)}" target="_blank">FAFSA PIN Access</apex:outputLink>
                     <apex:toolbar >
                        <apex:selectList value="{!selectedFile}" multiselect="false" size="1" rendered="{!IF(attachmentSize > 0, true, false)}">
                           <apex:selectOptions value="{!attachments}"/>
                        </apex:selectList>
                        <apex:commandButton value="Load" action="{!ParseLoans}" id="loadLoan" rendered="{!IF(attachmentSize > 0, true, false)}" 
                           oncomplete="init();" rerender="refreshme,quoterefresh" status="retrieveStatus" />
                        <apex:commandButton value="Retrieve Loans" oncomplete="location.reload(true);" id="retrieveloans" action="{!myCallout}"
                           rendered="{!IF(Lead__c.SSN_ENC__c != null && Lead__c.DOB__c != null && PinValid == true, true, false )}"
                           rerender="refreshme,quoterefresh" status="retrieveStatus"/>
                           <!--
                           && Lead__c.Queued_For_Loan_Retrieval__c == 'cleared', true, false)}" 
                           -->
                        <apex:commandButton value="Clear Loans" action="{!clearLoans}" id="clearLoans" oncomplete="init();" 
                           rendered="{!AND(NOT(ISNULL(loans)), loans.size > 0)}"
                           status="retrieveStatus"
                           rerender="refreshme,quoterefresh" />
                        <apex:outputPanel rendered="{!AND(NOT(ISNULL(loans)), loans.size > 0)}" style="color: white; font-weight: bold; font-size: 1.2em;">
                           Action To Take &nbsp;
                           <apex:selectList value="{!selectedAction}" html-data-id="selectedAction" multiselect="false" size="1">
                              <apex:selectOptions value="{!ActionOptions}"/>
                           </apex:selectList>
                           &nbsp; On &nbsp;
                           <apex:commandButton value="Selected" onClick="doAction();" html-data-id="selectedButton" status="retrieveStatus" oncomplete="init();"/>
                           &nbsp;
                           <apex:commandButton value="All" action="{!doAction}" rerender="refreshme,quoterefresh" status="retrieveStatus" oncomplete="init();"/>
                            <!--&nbsp; &nbsp;
                           <apex:commandButton value="Save Selected" action="{!doConsolidateLoansSelected}" rerender="refreshme,quoterefresh" status="retrieveStatus" oncomplete="init();"/>
                           -->
                          <!-- <apex:commandButton value="Show Selected Leads" action="{!processSelected}" rerender="table2"/>-->
                           
                        </apex:outputPanel>
                     </apex:toolbar>
                     <!--
                     <apex:outputText rendered="{!IF(Lead__c.Queued_For_Loan_Retrieval__c != 'cleared', true, false)}" style="color: #F33;"
                        value="An invalid response was received from the NSLDS service. The request has been queued for retrieval."/>
                     -->
                     <apex:actionStatus id="retrieveStatus">
                        <apex:facet name="stop">
                           <apex:repeat value="{!RetrieveErrors}" var="f"> 
                              <apex:outputText value="{!f}" style="color: #F33;"/><br/>
                           </apex:repeat>
                        </apex:facet>
                        <apex:facet name="start">
                           <apex:outputPanel >
                              <apex:image value="/img/loading32.gif" style="height: 15px;"/>
                              <apex:commandButton value="Processing..." status="mySaveStatus1" disabled="true"/>
                           </apex:outputPanel>
                        </apex:facet>
                     </apex:actionStatus>
                     <apex:pageBlockTable value="{!loans}" var="currentLoan" id="loansTable">
                     <!--
                        <apex:column headerValue="Included"
                              styleClass="{!IF(currentLoan.Interest_Estimated__c,  'sleEst', 'sleNone')}">
                           <apex:inputCheckbox html-data-id="included" value="{!currentLoan.Included__c}" 
                              html-data-available="{!IF(currentLoan.Principal_Balance__c > 0 || currentLoan.Interest_Balance__c > 0, true, false)}" 
                              disabled="{!IF(currentLoan.Principal_Balance__c > 0 || currentLoan.Interest_Balance__c > 0, FALSE, TRUE)}">
                              <apex:actionSupport action="{!removeLoan}" event="onchange" oncomplete="init();"
                                                                    rerender="quoterefresh,loansTable,refreshme,loandetailcont" />
                           </apex:inputCheckbox>
                        </apex:column>
                           <apex:inputCheckbox html-data-id="included" value="{!currentLoan.Included__c}" 
                              html-data-available="{!IF(currentLoan.Principal_Balance__c > 0 || currentLoan.Interest_Balance__c > 0, true, false)}" 
                              disabled="{!IF(currentLoan.Principal_Balance__c > 0 || currentLoan.Interest_Balance__c > 0, FALSE, TRUE)}" style="display:none;"/>
                     -->
                        <apex:column headerValue="Select"
                              styleClass="{!IF(currentLoan.Interest_Estimated__c==TRUE,  'sleEst', 'sleNone')}">
                           <apex:inputCheckbox html-data-id="included" disabled="{!IF(currentLoan.IsClosed__c == 'Open',false,true)}"
                              html-data-available="{!IF(currentLoan.IsClosed__c == 'Open' || currentLoan.IsClosed__c == 'Open Consolidation', true, false)}" html-data-loanid="{!currentLoan.Id}"/>
                        </apex:column>
                      
                      <!--<apex:column headerValue="ME">
                      <apex:inputCheckbox value="{!currentLoan.selected_Paye__c}" />
                       </apex:column>-->
                        <apex:column headerValue="Award ID" 
                              styleClass="{!IF(currentLoan.Interest_Estimated__c,  'sleEst', 'sleNone')}">
                           <apex:commandLink action="{!ChangeDebt}" rerender="debtDisplay" rendered="{!IF(currentLoan.Id != null, true, false)}">
                              <apex:param value="{!currentLoan.Id}" name="selectedDebt" assignTo="{!selectedDebt}"  />
                               
                              <apex:outputField value="{!currentLoan.Award_ID__c}" />
                           </apex:commandLink>
                        </apex:column>
                        <apex:column headerValue="Principal Balance"
                              styleClass="{!IF(currentLoan.Interest_Estimated__c==TRUE,  'sleEst', 'sleNone')}">
                           <apex:outputField value="{!currentLoan.Principal_Balance__c}" />
                        </apex:column>
                        <apex:column headerValue="Interest Balance"
                              styleClass="{!IF(currentLoan.Interest_Estimated__c,  'sleEst', 'sleNone')}">
                           <apex:outputField value="{!currentLoan.Interest_Balance__c}" />
                        </apex:column>
                        <apex:column headerValue="Monthly Payment"
                              styleClass="{!IF(currentLoan.Interest_Estimated__c,  'sleEst', 'sleNone')}">
                           <apex:outputField value="{!currentLoan.Monthly_Loan_Payment__c}" />
                        </apex:column>
                        <apex:column headerValue="Interest Rate"
                              styleClass="{!IF(currentLoan.Interest_Estimated__c,  'sleEst', 'sleNone')}">
                           <apex:outputField value="{!currentLoan.Interest_Rate__c}" />
                        </apex:column>
                        <apex:column headerValue="Rate Type"
                              styleClass="{!IF(currentLoan.Interest_Estimated__c,  'sleEst', 'sleNone')}">
                           <apex:outputField value="{!currentLoan.Interest_Rate_Type__c}" />
                        </apex:column>
                        <apex:column headerValue="Servicer"
                              styleClass="{!IF(currentLoan.Interest_Estimated__c,  'sleEst', 'sleNone')}">
                           <apex:outputField value="{!currentLoan.Creditor__r.Name}" />
                        </apex:column>
                        <apex:column headerValue="Loan Type"
                              styleClass="{!IF(currentLoan.Interest_Estimated__c,  'sleEst', 'sleNone')}">
                           <apex:outputField value="{!currentLoan.Loan_Type__c}" />
                        </apex:column>
                        <apex:column headerValue="Status"
                              styleClass="{!IF(currentLoan.Interest_Estimated__c,  'sleEst', 'sleNone')}">
                           <apex:outputField value="{!currentLoan.Loan_Status__c}"/>
                        </apex:column>
                        <apex:column headerValue="Status Date"
                              styleClass="{!IF(currentLoan.Interest_Estimated__c,  'sleEst', 'sleNone')}">
                           <apex:outputField value="{!currentLoan.Loan_Status_Date__c}"/>
                        </apex:column>
                     </apex:pageBlockTable>
                  </apex:pageBlockSection>
                  </apex:outputPanel>
                  <apex:pageBlockSection id="debtDisplay" columns="1">
                     <apex:dynamicComponent componentValue="{!displayDebt}" rendered="{!IF(selectedDebt !=null && NOT(isCreate),TRUE,FALSE)}"/>
                  </apex:pageBlockSection>
                  <!--
                  <apex:pageBlockSection columns="1" title="FAFSA Recovery Questions" >
                     <apex:outputLink value="{!URLFOR($Action.Lead__c.FAFSA_PIN_Access)}" target="_blank">FAFSA PIN Access</apex:outputLink>
                     <apex:outputText style="font-size: .7em;">
                        NOTE: Spaces are not allowed in recovery answers.
                     </apex:outputText>
                     <apex:pageBlockSectionItem >
                        <apex:outputLabel for="birth_city" value="What city were you born in?"/>
                        <apex:inputField id="birth_city" tabOrderHint="224" value="{!Lead__c.Birth_City__c}"/>
                     </apex:pageBlockSectionItem>
                     <apex:pageBlockSectionItem >
                        <apex:outputLabel for="school" value="What was the name of your elementary school?"/>
                        <apex:inputField id="school" tabOrderHint="225" value="{!Lead__c.Elementary_School__c}"/>
                     </apex:pageBlockSectionItem>
                     <apex:pageBlockSectionItem >
                        <apex:outputLabel for="pastime" value="What is your favorite pastime?"/>
                        <apex:inputField id="pastime" tabOrderHint="226" value="{!Lead__c.Favorite_Pastime__c}"/>
                     </apex:pageBlockSectionItem>
                     <apex:pageBlockSectionItem >
                        <apex:outputLabel for="birth_hospital" value="What is the name of the Hospital where you were born?"/>
                        <apex:inputField id="birth_hospital" tabOrderHint="227" value="{!Lead__c.Birth_Hospital__c}"/>
                     </apex:pageBlockSectionItem>
                     <apex:pageBlockSectionItem >
                        <apex:outputLabel for="favorite_color" value="What is your favorite color?"/>
                        <apex:inputField id="favorite_color" tabOrderHint="228" value="{!Lead__c.Favorite_Color__c}"/>
                     </apex:pageBlockSectionItem>
                     <apex:pageBlockSectionItem >
                        <apex:outputLabel for="maiden_name" value="What is your mother's maiden name?"/>
                        <apex:inputField id="maiden_name" tabOrderHint="229" value="{!Lead__c.MAIDEN__c}"/>
                     </apex:pageBlockSectionItem>
                     <apex:pageBlockSectionItem >
                        <apex:outputLabel for="dream_vacation" value="What is the location of your dream vacation?"/>
                        <apex:inputField id="dream_vacation" tabOrderHint="230" value="{!Lead__c.Dream_Vacation__c}"/>
                     </apex:pageBlockSectionItem>
                     <apex:pageBlockSectionItem >
                        <apex:outputLabel for="tv_show" value="What is your favorite TV show?"/>
                        <apex:inputField id="tv_show" tabOrderHint="231" value="{!Lead__c.TV_Show__c}"/>
                     </apex:pageBlockSectionItem>
                     <apex:pageBlockSectionItem >
                        <apex:outputLabel for="car" value="What brand was your first car?"/>
                        <apex:inputField id="car" tabOrderHint="232" value="{!Lead__c.First_Car__c}"/>
                     </apex:pageBlockSectionItem>
                  </apex:pageBlockSection>
                  -->
               </apex:pageBlock>
            </apex:form>
         </apex:tab>
         <apex:tab label="Loan Programs" name="quoteDetail" id="quoteDetail" onTabEnter="updateLastTab('quoteDetail');">
            <apex:form >
               <apex:pageBlock >
                  <apex:pageBlockButtons >
                     <apex:commandButton action="{!requote}" value="Recalculate Programs" rerender="quoterefresh" onComplete="init();" status="recalcstatus"/>
                     <apex:outputPanel styleClass="helpButton" html-data-id="loanhelp" id="loanHelp-_help" rendered="{!IF(loanQualificationText == null, false, true)}">
                        Program Qualification Details
                        <img src="/s.gif" class="helpOrb"/>
                        <apex:outputPanel html-data-id="loanhelp-text" style="display: none;">{!loanQualificationText}</apex:outputPanel>
                     </apex:outputPanel>
                     <apex:actionStatus id="recalcstatus">
                        <apex:facet name="stop">
                        </apex:facet>
                        <apex:facet name="start">
                           <apex:outputPanel >
                              <apex:image value="/img/loading32.gif" style="height: 15px;"/>
                              <apex:commandButton value="Processing..." status="mySaveStatus1" disabled="true"/>
                           </apex:outputPanel>
                        </apex:facet>
                     </apex:actionStatus>
                  </apex:pageBlockButtons>
                  <apex:outputPanel id="quoterefresh">
                  <apex:pageBlockSection id="quoteDetail">
                     <apex:outputField label="Current Loan Balance" value="{!Lead__c.Loan_Amount__c}"/>                       
                     <apex:outputField label="Loan Amount Borrowed" value="{!Lead__c.Loan_Amount_Borrowed__c}"/>
                     <apex:outputField label="Current Monthly Payment" value="{!Lead__c.Monthly_Student_Payment__c}"/>
                     <apex:outputField label="Current Annual Interest Rate" value="{!Lead__c.Interest_Rate__c}"/>                     
                  </apex:pageBlockSection>
                  <apex:pageBlockSection id="quoteList" columns="1">
                     <apex:actionFunction name="selectQuote" action="{!selectQuote}" rerender="quoterefresh"/>
                     <apex:pageBlockTable value="{!quotes}" var="cQuote" id="quotesTable" html-data-table="programTable">
                        <apex:column headerValue="Select" html-data-table="{!cQuote.ParentQuoteID}">
                           <apex:commandButton action="{!selectQuote}" rendered="{!IF(cQuote.ParentQuoteId=='SUM',true,false)}" 
                              rerender="quoterefresh"
                              disabled="{!IF(cQuote.Status!='Accepted' && hasQuote, TRUE, FALSE)}"
                              value="{!IF(cQuote.Status=='Accepted','Selected','Select')}"
                              onComplete="init();">
                              <apex:param name="quoteid" assignTo="{!selectedQuote}" value="{!cQuote.Id}"/>
                           </apex:commandButton>
                        </apex:column>
                        <apex:column value="{!cQuote.Repayment_Plan}" headerValue="Repayment Plan"/>
                        <apex:column headerValue="Loan Balance" >
                           <apex:outputText value="${0,number,###,###,###,###.##}" rendered="{!IF(cQuote.Loan_Amount != null, true, false)}">
                              <apex:param value="{!cQuote.Loan_Amount}" />
                           </apex:outputText>
                        </apex:column>
                        <apex:column headerValue="APR" value="{!cQuote.Interest_Rate}" />
                        <apex:column headerValue="Monthly Savings"> 
                           <apex:outputText value="${0,number,###,###,###,###.##}"
                              rendered="{!IF(cQuote.Savings != null, true, false)}"
                              styleClass="{!IF(cQuote.Savings != null && cQuote.Savings < 0,  'ndNeg', '')}">
                              <apex:param value="{!abs(cQuote.Savings)}" />
                           </apex:outputText>
                        </apex:column>
                        <apex:column headerValue="First Payment" >
                           <apex:outputText value="${0,number,###,###,###,###.##}"
                              rendered="{!IF(cQuote.First_Payment != null, true, false)}">
                              <apex:param value="{!cQuote.First_Payment}" />
                           </apex:outputText>
                        </apex:column>
                        <apex:column headerValue="Last Payment">
                           <apex:outputText value="${0,number,###,###,###,###.##}"
                              rendered="{!IF(cQuote.Last_Payment != null, true, false)}">
                              <apex:param value="{!cQuote.Last_Payment}" />
                           </apex:outputText>
                        </apex:column>
                        <apex:column headerValue="Total Loan Payments">
                           <apex:outputText value="${0,number,###,###,###,###.##}"
                              rendered="{!IF(cQuote.Total_Loan_Payment != null, true, false)}">
                              <apex:param value="{!cQuote.Total_Loan_Payment}" />
                           </apex:outputText>
                        </apex:column>
                        <apex:column headerValue="Total Interest Paid">
                           <apex:outputText value="${0,number,###,###,###,###.##}"
                              rendered="{!IF(cQuote.Total_Interest_Paid != null, true, false)}">
                              <apex:param value="{!cQuote.Total_Interest_Paid}" />
                           </apex:outputText>
                        </apex:column>
                        <apex:column headerValue="Forgiven Amount">
                           <apex:outputText value="${0,number,###,###,###,###.##}"
                              rendered="{!IF(cQuote.Loan_Forgiveness_Amount != null && cQuote.Loan_Forgiveness_Amount > 0, true, false)}">
                              <apex:param value="{!cQuote.Loan_Forgiveness_Amount}" />
                           </apex:outputText>
                        </apex:column>
                        <apex:column value="{!cQuote.Months_In_Repayment}" headerValue="Payment Months"/>
                        </apex:pageBlockTable>
                  </apex:pageBlockSection>
               </apex:outputPanel>
               </apex:pageBlock>
            </apex:form>
         </apex:tab>
         
         <apex:tab label="Payments" name="payments" id="payments" onTabEnter="updateLastTab('payments');" >
       <apex:form >
               <apex:pageBlock id="payments">
                  <apex:pageBlockButtons >
                     <apex:commandButton value="Save" action="{!Save}"/>
                     <apex:commandButton value="Cancel" action="{!Cancel}"/>
                  </apex:pageBlockButtons>
                  <apex:pageBlockSection columns="2" title="Payments" id="paymentblock" >
                     <c:LeadPayment Length_="10" Filter_="pending" Lead_Id="{!lead.Id}" Title_="Pending Payments" />
                     <!--<c:LeadPayment Length_="10" Filter_="completed" Lead_Id="{!lead.Id}" Title_="Processed Payments" />-->
                  </apex:pageBlockSection>
                  <apex:pageBlockSection columns="2" title="Program Choice">
                     <apex:inputField value="{!Lead__c.Processor__c}"/>
                     <apex:pageBlockSectionItem />
                     <apex:inputField value="{!Lead__c.Fee_Template_Meta__c}"/>
                     <apex:inputField value="{!Lead__c.Initial_Debit_Date__c}"/>
                     <apex:inputField value="{!Lead__c.Program_Length__c}"/>
                  </apex:pageBlockSection>
               </apex:pageBlock>
            </apex:form>
        </apex:tab>
         
         <apex:tab label="Bank/Debit/CC Information" name="bankInfo" id="bankInfo" onTabEnter="updateLastTab('bankInfo');" >
            <apex:form >
               <apex:pageBlock id="bankdetailcont">
                  <apex:pageBlockButtons >
                     <apex:commandButton value="Save" action="{!Save}" />
                     <apex:commandButton value="Cancel" action="{!Cancel}" />
                  </apex:pageBlockButtons>
                  <apex:pageBlockSection columns="2" title="Bank Account -- Debit/Credit Card">
                     <!--  Note: Repurposed Employer fields used, in SL, for CCard information -->                  
                     <apex:inputField tabOrderHint="301" value="{!Lead__c.Name_On_Account__c}" label="Name on Bank Account" />
                     <apex:inputField tabOrderHint="311" value="{!Lead__c.empname__c}" label="Name on Credit Card"/>  
                     <apex:inputField tabOrderHint="302" value="{!Lead__c.Bank_Name__c}"/>
                     <apex:PageBlockSectionItem />
                     <apex:inputField tabOrderHint="303" value="{!Lead__c.bankadd__c}"/>
                     <apex:inputField tabOrderHint="313" value="{!Lead__c.EMPADD1__c}" label="CC Billing Address Line 1" />                      
                     <apex:inputField tabOrderHint="304" value="{!Lead__c.bankcity__c}"/>
                     <apex:inputField tabOrderHint="314" value="{!Lead__c.EMPCITY__c}" label="CC Billing Address City" />                      
                     <apex:inputField tabOrderHint="305" value="{!Lead__c.BANKST__c}"/>
                     <apex:inputField tabOrderHint="315" value="{!Lead__c.EMPST__c}" label="CC Billing Address State" />                     
                     <apex:inputField tabOrderHint="306" value="{!Lead__c.bankpostal__c}"/>
                     <apex:inputField tabOrderHint="316" value="{!Lead__c.EMPPOSTAL__c}" label="CC Billing Address Postal Code" />                     
                  </apex:pageBlockSection>
                  <apex:pageBlockSection columns="2" title="Account Information">
                     <apex:inputField tabOrderHint="321" value="{!Lead__c.Routing_Number__c}"/>
                     <apex:inputField tabOrderHint="331" value="{!Lead__c.Debit_Card_Number__c}"/>                     
                     <apex:inputField tabOrderHint="322" value="{!Lead__c.Account_Number_Enc__c}"/>
                     <apex:inputField tabOrderHint="332" value="{!Lead__c.CVV_Code__c}"/>                     
                     <apex:inputField tabOrderHint="323" value="{!Lead__c.Account_Type__c}"/>
                     <apex:inputField tabOrderHint="333" value="{!Lead__c.Debit_Card_Expires__c}"/>                     
                  </apex:pageBlockSection>
                  <!--<apex:pageBlockSection columns="2" title="Program Choice">
                     <apex:inputField value="{!Lead__c.Processor__c}"/>
                     <apex:pageBlockSectionItem />
                     <apex:inputField value="{!Lead__c.Fee_Template_Meta__c}"/>
                     <apex:inputField value="{!Lead__c.Initial_Debit_Date__c}"/>
                     <apex:inputField value="{!Lead__c.Program_Length__c}"/>
                  </apex:pageBlockSection>-->
               </apex:pageBlock>
            </apex:form>
         </apex:tab>
         <apex:tab label="Financial Profile" name="finTab" id="finTab" onTabEnter="updateLastTab('finProfile');" >
            <c:FinancialProfileComponent clientId="{!Lead__c.Id}"/>
         </apex:tab>
         <apex:tab label="Employer & References" name="refTab" id="refTab" onTabEnter="updateLastTab('refTab');" >
            <c:Employer cid="{!Lead__c.Id}"/>
            <apex:form >
               <apex:pageBlock id="references" title="References">
                  <apex:pageBlockButtons >
                     <apex:commandButton value="Save Reference" action="{!saveContact}" rerender="references,pageMessage"/>
                     <apex:commandButton value="Cancel" action="{!newContact}" rerender="references,pageMessage" rendered="{!IF(selectedContact==null,FALSE,TRUE)}"/>
                  </apex:pageBlockButtons>
                  <apex:pageBlockTable value="{!contacts}" var="c" rendered="{!hasRefs}">
                     <apex:column >
                        <apex:commandLink action="{!editReference}" value="Edit" rerender="references,pageMessage">
                           <apex:param name="refId" value="{!c.Id}" assignTo="{!selectedContact}"/>
                        </apex:commandLink>
                        &nbsp; | &nbsp;
                        <apex:commandLink action="{!deleteContact}" rerender="references,pageMessage" value="Delete">
                           <apex:param name="refId" value="{!c.Id}" assignTo="{!selectedContact}"/>
                        </apex:commandLink>
                     </apex:column>
                     <apex:column headerValue="Name" value="{!c.FirstName} {!c.LastName}"/>
                     <apex:column headerValue="Address" value="{!c.MailingStreet}; {!c.MailingCity}, {!c.MailingState} {!c.MailingPostalCode}"/>
                     <apex:column headerValue="Phone" value="{!c.Phone}" />
                     <apex:column headerValue="Email" value="{!c.Email}"/>
                     <apex:column headerValue="Relationship" value="{!c.Relationship__c}"/>
                  </apex:pageBlockTable>
                  <apex:pageBlockSection >
                     <apex:inputText label="First Name" value="{!contact.FirstName}" />
                     <apex:inputText label="Last Name" value="{!contact.LastName}" />
                     <apex:inputTextArea label="Address" value="{!contact.Street}"/>
                     <apex:inputText label="City" value="{!contact.City}"/>
                     <apex:inputText label="State" value="{!contact.State}"/>
                     <apex:inputText label="Postal Code" value="{!contact.Postal}"/>
                     <apex:inputText label="Country" value="{!contact.Country}"/>
                     <apex:inputText label="Email" value="{!contact.Email}"/>
                     <apex:inputText label="Phone" value="{!contact.Phone}" onblur="formatPhone(this);"/>
                     <apex:inputText label="Relationship" value="{!contact.Relationship}"/>
                  </apex:pageBlockSection>
               </apex:pageBlock>
            </apex:form>
         </apex:tab>
         <apex:tab label="Activity" name="activityTab" id="activityTab" onTabEnter="updateLastTab('activityTab');" >
            <apex:outputPanel id="activityPanel">
            <apex:include pageName="ActivityPage"/>
            </apex:outputPanel>
         </apex:tab>
         <apex:tab label="Documents" name="attachmentTab" id="attachmentTab" onTabEnter="updateLastTab('attachmentTab');" >
            <apex:relatedList list="NotesAndAttachments" id="attachementBlock"/>
         </apex:tab>    
      </apex:tabPanel>
      <c:DatePicker />
</apex:page>


<<<-------studentleadcontroller----------->>>>>>>>

/*
*  Controller class for the Student Loan lead visualforce page
*  page/studentLead.page
*/

public without sharing class StudentLeadController
{
   public Lead__c lead { get; set; }
   public List<Lead_Debt__c> Loans { get; set; }
   public Id selectedFile { get; set; }
   public Id selectedDebt { get; set; }
   public Boolean isCreate { get; set; }
   public Integer attachmentSize { get; set; }
   public Boolean newQuotesNeeded { get; set; }
   public List<New_Quote__c> nquotes { get; set; }                // model layer
   public List<quoteWrapper> quotes { get; set; }                 // display
   private List<PDMap__c> pds = new List<PDMap__c>();   
   public String quoteStatus { get; set; }
   public Boolean isMarried { get; set; }
   public Boolean isSeparateTax { get; set; }
   private ApexPages.StandardController controller = null;
   private Id quoteRt { get; set; }
   private Id contactRt { get; set; }
   private List<String> nsldsErrors = new List<String>();
   public FSWrapper leadDetailTop { get; set; }
   public FSWrapper leadDetailBottom { get; set; }
   public String urlFromPage { get; set; }
   public String lastActivityLink { get; set; }
   public String lastActivityText { get; set; }
   public Boolean updated { get; set; }
   public String selectedQuote { get; set; }
   public Boolean hasQuote { get; set; }
   public List<Contact> contacts { get; set; }
   public Id selectedContact { get; set; }
   public Id loanId { get; set; }
   public String selectedLoans { get; set; }
   public String selectedAction { get; set; }
   public Map<String,String> formulaFields { get; set; }
   public Map<String,Boolean> formulaFieldHasValue { get; set; }

   public Boolean getPinValid()
   {
      Boolean valid = false;
      if ( this.lead.FAFSA_PIN_ENC__C != null && this.lead.FAFSA_PIN_ENC__c.length() == 4 )
      {
         valid = true;
      }
      return valid;
   }
   
   
   
   

   public Reference contact { get; set; }

   public Boolean getQSize()
   {
      return this.quotes.size() > 0;
   }

   public String getLoanQualificationText()
   {
      List<String> help = new List<String>();

      Map<String, Boolean> Defaulted = new Map<String, Boolean>{
         'DB' => true,
         'DC' => true,
         'DD' => true,
         'DF' => true,
         'DK' => true,
         'DL' => true,
         'DN' => true,
         'DO' => true,
         'DP' => true,
         'DR' => true,
         'DS' => true,
         'DT' => true,
         'DU' => true,
         'DW' => true,
         'DX' => true,
         'DZ' => true,
         'XD' => true
      };

      // check if any loans closed through consolidation simulation are in default
      Boolean openInDefault = false;
      Boolean consolidatedDefault = false;
      for ( Lead_Debt__c ld : this.Loans )
      {
         // check if any currently open loans are in default
         if ( Defaulted.get(ld.Loan_Status_Code__c) == true &&
            openInDefault == false &&
            ld.IsClosed__c == 'Open')
         {
            openInDefault = true;
            help.add('Must go through rehab or consolidation due to loans in default.');
         }

         if ( Defaulted.get(ld.Loan_Status_Code__c) == true &&
            consolidatedDefault == false &&
            ld.IsClosed__c == 'Closed Through Simulation')
         {
            consolidatedDefault = true;
            help.add('Consolidations containing defaulted loans must go through an income based repayment program.');
         }
      }

      // check if income information is completed
      if ( this.lead.State__c == null ||
         this.lead.Estimated_AGI__c == null ||
         this.lead.Household_Size__c == null )
      {
         System.Debug('adding lead stuff');
         help.add('Income based repayment plans require that State, Estimated AGI, and Household Size be provided.');
      }

      String helptext = String.join(help,'<br/>');

      return helptext;

      //return String.join('\\n',help);
   }
   
   public class quoteWrapper
   {
      public Id id { get; set; }
      public Decimal Loan_Amount { get; set; }
      public Decimal Interest_Rate { get; set; }
      public Decimal First_Payment { get; set; }
      public Decimal Last_Payment { get; set; }
      public Decimal Total_Loan_Payment { get; set; }
      public Decimal Total_Interest_Paid { get; set; }
      public Integer Months_In_Repayment { get; set; }
      public Decimal Loan_Forgiveness_Amount { get; set; }
      public String  Status { get; set; }
      public String  Repayment_Plan { get; set; }      
      public Decimal Savings { get; set; }
      public String ParentQuoteID { get; set; }

      public quoteWrapper( New_Quote__c nq )
      {
         this.id = nq.id;
         this.loan_amount = nq.loan_amount__c;
         this.interest_rate = nq.interest_rate__c;
         this.first_payment = nq.first_payment__c;
         this.last_payment = nq.last_payment__c;
         this.total_loan_payment = ( nq.total_interest_paid__c == null ? 0 : nq.total_interest_paid__c ) + 
            ( nq.total_principal_paid__c == null ? 0 : nq.total_principal_paid__c );         
         this.total_interest_paid = nq.total_interest_paid__c;
         this.months_in_repayment = Integer.valueOf(nq.months_in_repayment__c);
         this.loan_forgiveness_amount = nq.loan_forgiveness_amount__c;
         this.status = nq.status__c;
         this.repayment_plan = nq.repayment_plan__c;
         this.ParentQuoteID = nq.ParentQuoteID__c;
      }

      public quoteWrapper() {}
   }

   public class Reference
   {
      public String FirstName { get; set; }
      public String LastName { get; set; }
      public String Street { get; set; }
      public String City { get; set; }
      public String State { get; set; }
      public String Postal { get; set; }
      public String Country { get; set; }
      public String Email { get; set; }
      public String Phone { get; set; }
      public String Relationship { get; set; }
      public String Id { get; set; }
   }

   public Boolean getHasRefs()
   {
      return this.contacts.size() > 0;
   }

   public class FSWrapper
   {
      public String label { get; set; }
      public List<Schema.FieldSetMember> fields { get; set; }

   }
   
   public Component.Apex.OutputPanel getLeadButtons() 
   {
      Component.Apex.OutputPanel op = new Component.Apex.OutputPanel();
      try 
      {
         NuDebtCustom2 custom = CustomFactory.getCustomInstance2();
         op = custom.getLeadButtons(this.lead);
      }
      catch (Exception ex) 
      {
         ApexPages.addMessages(ex);
      }
      return op;
   }

   public PageReference Convert()
   {
      if ( this.lead.isConverted__c == true )
      {
         this.addMessage( ApexPages.severity.WARNING, 'Lead is already converted.' );

         return ApexPages.currentPage();
      }

      LeadToClient l2c = CustomFactory.getLeadConversionStudent();

      try 
      {
         l2c.preCopy( lead.Id );
         l2c.copyLeadToClient( lead.Id );
         l2c.postCopy( lead.Id, l2c.client );
      } catch ( Exception ex )
      {
         ApexPages.addMessages(ex);
      }

      if ( l2c.client != null )
      {
         this.lead.isConverted__c = true;
         this.lead.Client__c = l2c.client;
         this.lead.Lead_Status__c = 'Converted';
         update this.lead;
         this.addMessage( ApexPages.severity.INFO, 'Lead successfully converted.' );

         PageReference pr = ApexPages.currentPage();
         return pr;
      }
      else
      {
         this.addMessage( ApexPages.severity.ERROR, 'Error converting lead' );
         return null;
      }
   }

   public String selectedTab
   {
      get
      {
         if ( ApexPages.currentPage().getParameters().get('tab') != null )
         {
            this.selectedTab = ApexPages.currentPage().getParameters().get('tab');
         }
         else if ( this.lead.Last_Viewed__c != null )
         {
            this.selectedTab = this.lead.Last_Viewed__c;
         }
         else if ( this.selectedTab == null )
         {
            this.selectedTab = 'activityTab';
         }

         return this.selectedTab;
      }
      set
      {
         if ( value != null )
         {
            this.selectedTab = value;
         }
         else
         {
            this.selectedTab = 'activityTab';
         }
      }
   }
   
   public List<SelectOption> attachmentList = new List<SelectOption>();

   /*
   *  Constructor
   */
   public StudentLeadController ( ApexPages.StandardController ctrl )
   {
      this.controller = ctrl;
      this.lead = (Lead__c)ctrl.getRecord();
      this.updated = false;

      List<Lead__c> temp = [select Last_Viewed__c, FAFSA_PIN__c, FAFSA_PIN_ENC__c, isConverted__c from Lead__c where Id = :this.lead.Id];

      if(temp.size() > 0 )
      {
         this.lead.Last_Viewed__c = temp[0].Last_Viewed__c;
         this.lead.isConverted__c = temp[0].isConverted__c;

         if ( temp[0].FAFSA_PIN__c != null && 
              temp[0].FAFSA_PIN__c != '' &&
            ( temp[0].FAFSA_PIN_ENC__c == null || temp[0].FAFSA_PIN_ENC__c == '' ) )
         {
            this.lead.FAFSA_PIN_ENC__c = temp[0].FAFSA_PIN__c;
            this.lead.FAFSA_PIN__c = null;
            this.updated = true;
         }
      }

      this.isCreate = false;
      this.isMarried = false;

      if ( this.lead.Marital_Status__c == 'Married' ||
           this.lead.Marital_Status__c == 'Separated' )
      {
         this.isMarried = true;
      }

      this.isSeparateTax = false;
      if ( this.lead.Taxes_Filed__c == 'Separately' ) 
      {
         this.isSeparateTax = true;
      }

      this.loadFieldSets();

      this.quoteStatus = 'Pending';

      this.quoteRt = Schema.SObjectType.New_Quote__c.getRecordTypeInfosByName().get('Student Loan').getRecordTypeId();
      
      this.contactRt = Schema.SObjectType.Contact.getRecordTypeInfosByName().get('Reference').getRecordTypeId();
      
      this.LoadData();
   }

   private void loadFieldSets()
   {
      SLE_Display_Fieldsets__c settings = SLE_Display_Fieldsets__c.getInstance();

      if ( settings.Lead_Lead_Detail_Top__c != null )
      {
         try
         {
            System.Debug('HERE 1');
            this.leadDetailTop = new FSWrapper();
            System.Debug('HERE 1');
            Schema.FieldSet FSTop = Schema.SObjectType.Lead__c.fieldSets.getMap().get(settings.Lead_Lead_Detail_Top__c);
            System.Debug('HERE 1');
            if ( FSTop != null )
            {
               System.Debug('HERE 1');
               this.leadDetailTop.label = FSTop.getLabel();
               System.Debug('HERE 2');
               this.leadDetailTop.fields = FSTop.getFields();
               System.Debug('HERE 3');
               this.LoadLeadFields(FSTop);
               System.Debug('HERE 4');
            }
         }
         catch (Exception ex)
         {
            System.Debug(ex.getMessage());
         }
      }

      if ( settings.Lead_Lead_Detail_Bottom__c != null )
      {
         try
         {
            this.leadDetailBottom = new FSWrapper();
            Schema.FieldSet FSBottom = Schema.SObjectType.Lead__c.fieldSets.getMap().get(settings.Lead_Lead_Detail_Bottom__c);
            if ( FSBottom != null )
            {
               this.leadDetailBottom.label = FSBottom.getLabel();
               this.leadDetailBottom.fields = FSBottom.getFields();
               this.LoadLeadFields(FSBottom);
            }
         }
         catch (Exception ex)
         {
            System.Debug(ex.getMessage());
         }
      }
   }

   private void LoadLeadFields( Schema.FieldSet FS )
   {
      String query = 'SELECT ID, ';
      List<String> fields = new List<String>();
      
      for( Schema.FieldSetMember fsm : FS.getFields() )
      {
         String item = fsm.fieldPath;
         System.Debug(' adding '+item);
         fields.add(item);
      }
      query += String.join(fields,',') + ' FROM Lead__c WHERE ID=\''+this.lead.ID+'\'';
      System.Debug('query '+query);
      List<Lead__C> temp = Database.query(query);
      if(temp.size() > 0)
      {
         Lead__c x = temp.get(0);
         System.Debug(' here with >'+x+'<');
         for( Schema.FieldSetMember fsm : FS.getFields() )
         {
            System.Debug('adding >'+fsm.fieldPath+'< of >'+x.get(fsm.fieldPath)+'<');
            try {
               this.lead.put(fsm.fieldPath, x.get(fsm.fieldPath));
            } catch ( Exception ex )
            {
               if ( this.formulaFields == null )
               {
                  this.formulaFields = new Map<String,String>();
                  this.formulaFieldHasValue = new Map<String,Boolean>();
               }
               this.formulaFields.put(String.valueOf(fsm.fieldPath), String.valueOf(x.get(fsm.fieldPath)));
               this.formulaFieldHasValue.put(String.valueOf(fsm.fieldPath), (x.get(fsm.fieldPath) != null) );
            }
         }
         System.Debug('>here with >'+this.formulaFieldHasValue+'<');
      }
   }

   private Integer wLoadContacts()
   {
      Integer irtn = 0;
      try
      {
      this.contacts = [SELECT
                           FirstName,
                           Id,
                           LastName,
                           Email,
                           MailingCity,
                           MailingState,
                           MailingCountry,
                           MailingPostalCode,
                           MailingStreet,
                           Phone,
                           RecordTypeId,
                           Lead__c,
                           Relationship__c
                           FROM
                           Contact
                           WHERE RecordTypeId = :this.contactRt
                           AND Lead__c = :this.lead.Id
                         ];
        irtn = this.contacts.size();
      }
      catch ( Exception ex )
      {
        System.Debug( 'StudentLeadController().wLoadContacts() Exception= '+ex.getMessage());
        irtn = 0;
      }
      return irtn;                            
   }        
    
   private void wLoadAttachments()
   {
    List<Attachment> leadattach = [ SELECT
                                    ID,
                                    Name,
                                    CreatedDate,
                                    Body,
                                    ContentType
                                    FROM
                                    Attachment
                                    WHERE
                                    ParentId = :this.lead.ID
                                    ORDER BY CreatedDate DESC
                                                ];
      this.attachmentList.clear();
    for ( Attachment a : leadattach )
    {
      try {
      String content = a.body.toString();
      Pattern fileMatch = Pattern.compile('File\\s+Source\\:');
      Matcher fileMatcher = fileMatch.matcher(content);
      if ( fileMatcher.find() )
      {
         this.attachmentList.add( new SelectOption( a.Id, a.Name ) );
      }
      } catch (Exception ex) { System.Debug(ex); }
    }
    this.attachmentSize = this.attachmentList.size();
    
   }

   public List<SelectOption> getActionOptions()
   {
      return new List<SelectOption>{
         new SelectOption('Consolidate','Consolidate')
      };
   }

  private Integer wLoadLoans()
   {
      Integer irtn = 0;
      
      try 
      {
       this.Loans =  [SELECT
                      ID,
                      Award_ID__c,
                      Principal_Balance__c,
                      Interest_Balance__c,
                      Interest_Rate__c,
                      Interest_Rate_Type__c,
                      Monthly_Loan_Payment__c,
                      Included__c,
                      Interest_Estimated__c,                    
                      Creditor__r.Name,
                      Loan_Status_Date__c,
                      Loan_Status_Code__c,
                      Loan_Type__c,
                      Verified__c,
                      Loan_Amount__c,
                      Loan_Status__c,
                      Disbursed__c,
                      Loan_Code__c,
                      Lead__c,
                      CanICR__c,
                      CanIBR__c,
                      CanExt__c,
                      CanPayE__c,
                      IsDefault__c,
                      IsClosed__c,
                      RequireIB__c,
                      Loan_Date__c,
                      Loan_Paid__c,
                      Consolidation__c,
                      selected_Paye__c
                      FROM
                      Lead_Debt__c
                      WHERE
                      Lead__r.ID = :this.lead.ID
                      ORDER BY Included__c DESC, Principal_Balance__c DESC, Interest_Balance__c DESC
                    ];
        irtn = this.Loans.size();
      }
      catch ( Exception ex )
      {
        System.Debug( 'StudentLeadController().wLoadLoans() Exception= '+ex.getMessage());
        irtn = 0;
      }
      return irtn;      
   }
   
   private void wDeleteLoans()
   {
      try
      {
        if ( this.Loans.size() > 0 )
        {
           delete this.Loans;
           this.Loans = new List<Lead_Debt__c>();
         }
      }
      catch (Exception ex)
      {
        System.Debug( 'StudentLeadController().wDeleteLoans Exception= '+ex.getMessage());
      }     
      wDeletePDs();
   }

   private integer wLoadQuotes()
   {
      System.Debug('loading Quotes');
      Integer irtn = 0; 
      try 
      {
         System.Debug('getting quotes using >'+this.quoteRt+'< and >'+this.lead.Id+'<');
         System.Debug('getting quotes using >'+this.quoteRt.getSObjectType()+'< and >'+this.lead.Id.getSObjectType()+'<');
        this.nquotes = [SELECT                         
                        ID,
                              Repayment_Plan__c,                                 
                            Loan_Amount__c,                
                            Interest_Rate__c,              
                                              Total_Principal_Paid__c,                       
                            Total_Interest_Paid__c,
                            Total_Monthly_Payment__c,                       // Orignal Monthly payment for all debt in this program/quote                         
                            First_Payment__c,              
                            Last_Payment__c,
                            Monthly_Payment__c,
                        Lead__c,
                        Months_In_Repayment__c,                            
                        Loan_Forgiveness_Amount__c,    
                        Status__c,                     
                        Multiple_Servicers__c,
                        Has_Direct__c,                        
                        Has_FFEL__c,
                        ParentQuoteID__c               
                        FROM                           
                        New_Quote__c                   
                        WHERE                          
                        RecordTypeId = :this.quoteRt   
                        AND Lead__c = :this.lead.Id    
                   ];
        irtn = this.nquotes.size();
      }
      catch ( Exception ex )
      {
        System.Debug( 'StudentLeadController().wLoadQuotes() Exception= '+ex.getMessage());
        irtn = 0;
      }
      System.Debug('Got >'+irtn+'< quotes');
      return irtn;            
  }
  
   private void wDeleteQuotes()
   {
      try 
      {
         if ( this.nquotes.size() > 0 )
         {
            delete this.nquotes;
            this.nquotes = new List<New_Quote__c>();
         }
      }
      catch (Exception ex)
      {
        System.Debug( 'StudentLeadController().wDeleteQuotes Exception= '+ex.getMessage());
      }           
      wDeletePDs();
   }
   
   private void wBuildQuoteWrappers()
   {
     // check if quotes are accurate and inline with current information
    this.newQuotesNeeded = true;
    this.hasQuote = false;

    List<String> order = new List<String>{
      StudentLoanCalcUtilities.getNameStd(),
      StudentLoanCalcUtilities.getNameStdExt(),
      StudentLoanCalcUtilities.getNameGrd(),
      StudentLoanCalcUtilities.getNameGrdExt(),
      StudentLoanCalcUtilities.getNameIBR(),
      StudentLoanCalcUtilities.getNameICR(),
      StudentLoanCalcUtilities.getNamePayE()
    };

    List<New_Quote__c> copyQuotes = this.nquotes.clone();
    this.nquotes = new List<New_Quote__c>();

    for ( string pgm : order )
    {
      Integer qSize = copyQuotes.size();
      for(Integer indx = 0; indx < qSize; indx++ )
      {
         if ( copyQuotes.get(indx).Repayment_Plan__c == pgm )
         {
            this.nquotes.add(copyQuotes.get(indx));
            copyQuotes.remove(indx);
            indx--;
            qSize--;
         }
      }
    }
    
   this.quotes = new List<quoteWrapper>();
   Map<String,List<quoteWrapper>> qmap = new Map<String,List<quoteWrapper>>();
   for ( New_Quote__c q : this.nquotes )
   {
      if( q.Status__c == 'Accepted' )
      {
         this.hasQuote = true;
      }                                            

      if ( q.ParentQuoteId__c != null && 
         qmap.get(q.ParentQuoteId__c) != null )
      {
         qmap.get(q.ParentQuoteId__c).add( new quoteWrapper(q) );
      }
      else if( q.ParentQuoteId__c != null )
      {
         qmap.put(q.ParentQuoteId__c, new List<quoteWrapper>() );
         qmap.get(q.ParentQuoteId__c).add( new quoteWrapper(q) );
      }
      else if( q.ParentQuoteId__c == null &&
         qmap.get(q.Id) != null )
      {
         qmap.get(q.Id).add( new quoteWrapper(q) );
      }
      else if( q.ParentQuoteId__c == null )
      {
         qmap.put(q.Id, new List<quoteWrapper>() );
         qmap.get(q.Id).add( new quoteWrapper(q) );
      }
   }

   for ( New_Quote__c cQuote : this.nquotes )
   {
      if ( cQuote.ParentQuoteId__c != null )
      {
         continue;
      }
      String key = cQuote.Id;
      List<quoteWrapper> children = new List<quoteWrapper>();
      quoteWrapper qp;
      quoteWrapper summary = new quoteWrapper();
      Map<Integer,Decimal> lastPayments = new Map<Integer,Decimal>();

          for ( quoteWrapper qw : qmap.get(key) )
          {
             Decimal fr = ( qw.Loan_Forgiveness_Amount == null ? 0 : qw.Loan_Forgiveness_Amount );
             if ( qw.Id == key )
             {
                qp = qw;
                if ( lastPayments.get(qp.Months_In_Repayment) != null )
                {
                   lastPayments.put(qp.Months_In_Repayment,
                      lastPayments.get(qp.Months_In_Repayment) +
                      ( qp.Last_Payment == null ? qp.First_Payment : qp.Last_Payment ) );
                }
                else
                {
                   lastPayments.put(qp.Months_In_Repayment,
                      ( qp.Last_Payment == null ? qp.First_Payment : qp.Last_Payment ) );
                }
    
                summary.id = qp.Id;
                summary.Status = qp.Status;
                summary.Loan_Amount = ( summary.Loan_Amount == null ? 
                   qp.Loan_Amount : summary.Loan_Amount + qp.Loan_Amount );
                summary.First_Payment = ( summary.First_Payment == null ? 
                   qp.First_Payment : summary.First_Payment + qp.First_Payment );
                summary.Total_Loan_Payment = ( summary.Total_Loan_Payment == null ?
                   qp.Total_Loan_Payment : summary.Total_Loan_Payment + qp.Total_Loan_Payment );
                summary.Total_Interest_Paid = ( summary.Total_Interest_Paid == null ?
                   qp.Total_Interest_Paid : summary.Total_Interest_Paid + qp.Total_Interest_Paid );
                summary.Months_In_Repayment = ( summary.Months_In_Repayment == null ?
                   qp.Months_In_Repayment : ( summary.Months_In_Repayment > qp.Months_In_Repayment ?
                   summary.Months_In_Repayment : qp.Months_In_Repayment ) );
                summary.Loan_Forgiveness_Amount = ( summary.Loan_Forgiveness_Amount == null ?
                   fr : summary.Loan_Forgiveness_Amount + fr );
                summary.Repayment_Plan = ( summary.Repayment_Plan == null ?
                   qp.Repayment_Plan : qp.Repayment_Plan+'/'+summary.Repayment_Plan );
             }
             else
             {
                children.add(qw);
    
                if ( lastPayments.get(qw.Months_In_Repayment) != null )
                {
                   lastPayments.put(qw.Months_In_Repayment,
                      lastPayments.get(qw.Months_In_Repayment) +
                      ( qw.Last_Payment == null ? qw.First_Payment : qw.Last_Payment ) );
                }
                else
                {
                   lastPayments.put(qw.Months_In_Repayment,
                      ( qw.Last_Payment == null ? qw.First_Payment : qw.Last_Payment ) );
                }
    
                summary.Loan_Amount = ( summary.Loan_Amount == null ? 
                   qw.Loan_Amount : summary.Loan_Amount + qw.Loan_Amount );
                summary.First_Payment = ( summary.First_Payment == null ? 
                   qw.First_Payment : summary.First_Payment + qw.First_Payment );
                summary.Total_Loan_Payment = ( summary.Total_Loan_Payment == null ?
                   qw.Total_Loan_Payment : summary.Total_Loan_Payment + qw.Total_Loan_Payment );
                summary.Total_Interest_Paid = ( summary.Total_Interest_Paid == null ?
                   qw.Total_Interest_Paid : summary.Total_Interest_Paid + qw.Total_Interest_Paid );
                summary.Months_In_Repayment = ( summary.Months_In_Repayment == null ?
                   qw.Months_In_Repayment : ( summary.Months_In_Repayment > qw.Months_In_Repayment ?
                   summary.Months_In_Repayment : qw.Months_In_Repayment ) );
                summary.Loan_Forgiveness_Amount = ( summary.Loan_Forgiveness_Amount == null ?
                   fr : summary.Loan_Forgiveness_Amount + fr );
                summary.Repayment_Plan = ( summary.Repayment_Plan == null ?
                   qw.Repayment_Plan : summary.Repayment_Plan+'/'+qw.Repayment_Plan );
             }
          }
          

      Integer maxMonths = 0;
      for ( Integer months : lastPayments.keySet() )
      {
         if ( months > maxMonths )
         {
            maxMonths = months;
         }
      }

      if ( children.size() > 0 )
      {
         
         summary.ParentQuoteID = 'SUM';
         summary.Savings = ( this.lead.Monthly_Student_Payment__c == null ? 0 : this.lead.Monthly_Student_Payment__c ) - 
            ( summary.First_Payment == null ? 0 : summary.First_Payment );
         summary.Last_Payment = lastPayments.get(maxMonths);
         this.quotes.add(summary);
         this.quotes.add(qp);
         this.quotes.addAll(children);
      }
      else if( qp != null )
      {
         qp.Savings = ( this.lead.Monthly_Student_Payment__c == null ? 0 : this.lead.Monthly_Student_Payment__c ) - 
            ( qp.First_Payment == null ? 0 : qp.First_Payment );
         qp.Last_Payment = ( lastPayments.size() > 0 && lastPayments.get(maxMonths) != null ? lastPayments.get(maxMonths) : null );
         qp.ParentQuoteId = 'SUM';
         this.quotes.add(qp);
      }
      qmap.remove(key);
   }

   }  
   
   private Integer wLoadPDs()
   {
      Integer irtn = 0; 
      try 
      {
        this.pds = [ SELECT id, Lead__c from PDMap__c where Lead__c = :this.lead.Id ];       
       irtn = this.pds.size();
      }
      catch ( Exception ex )
      {
        System.Debug( 'StudentLeadController().wLoadPDs() Exception= '+ex.getMessage());
        irtn = 0;
      }
      return irtn;            
   }

   private void wDeletePDs( )
   {
      try
      {
        if ( this.pds.size() > 0 )
        {
           delete pds;
           pds = new List<PDMap__c>();
        }
      }
      catch (Exception ex)
      {
        System.Debug( 'StudentLeadController().wDeletePDs() Exception= '+ex.getMessage());
      }     
   }

   private void LoadData()
   {
      this.contact = new Reference();

      wLoadContacts();
      
      wLoadAttachments();      
      
      wLoadLoans();
         
      wLoadQuotes();

      wLoadPDs();

      wBuildQuoteWrappers();
   }

   public List<SelectOption> getAttachments()
   {
      return this.attachmentList;
   }

   public PageReference ClearLoans()
   {
     this.nsldsErrors.clear();  
            
     this.wDeleteLoans();
     this.wDeleteQuotes();       
     this.wBuildQuoteWrappers();                                    // wrappers aren't object based, thus need to wipe them
     
     lead.Loan_Amount_Borrowed__c = 0;
     lead.Interest_Rate__c = 0;
     lead.Loan_Amount__c = 0;
     lead.Monthly_Student_Payment__c = 0;
     lead.CALC_Monthly_Student_Payment__c = 0;
     update lead;
     
     return null;
   }

  public PageReference RetrieveAndParseLoans()
  {
    this.nsldsErrors.clear();   
    NSLDSLogin nl = new NSLDSLogin( this.lead.Id );
    Id df = nl.LoadLoans(); 

    if ( df != null )
    {
      this.wDeleteLoans();
      this.wDeleteQuotes();                    // delete the old quotes
        
      ParseLoanData pld = new ParseLoanData(this.lead, df, this.Loans );
      Integer parsed = pld.load();
      update this.lead;      
      if ( parsed > 0 )
      {
         this.wLoadLoans();
         PDMaps pdm = new PDMaps( this.Loans, this.lead ); // creates pdmap objects
         this.wLoadPDs();
         this.wLoadQuotes();
         StudentLoanCalc slpc = new StudentLoanCalc( this.nquotes, this.lead );

         this.wBuildQuoteWrappers();
      }
      else
      {
         this.nsldsErrors = nl.retrieveErrors;
      }
    }
    else
    {
      this.nsldsErrors = nl.retrieveErrors;
    }    
    
    return null;
  }
   
  public PageReference ParseLoans()
  {
    this.nsldsErrors.clear();   
    
    this.wDeleteLoans();
    this.wDeleteQuotes();       

    ParseLoanData pld = new ParseLoanData(this.lead, this.selectedFile, this.loans );
    Integer parsed = pld.load();
    update this.lead;      
    if ( parsed > 0 )
    {
      this.wLoadLoans();
      
      try {
      PDMaps pdm = new PDMaps( this.Loans, this.lead ); // creates pdmap objects
      this.wLoadPDs();
      this.wLoadQuotes();
      StudentLoanCalc slpc = new StudentLoanCalc( nquotes, lead );
      } catch (Exception ex)
      {
         System.Debug('HERE >'+ex+'<');
         //ApexPages.addmessage(new ApexPages.message(ApexPage , msg ));
      }

      this.wBuildQuoteWrappers();
    }

    return null;
  }      
   
   
   public List<String> getRetrieveErrors()
   {
      return this.nsldsErrors;
   }
      

   public PageReference changeDebt()
   {
      return null;
   }

   public PageReference Save ()
   {
      try
      {
        
         if ( this.updated )
         {
            this.updated = false;
         }
         update this.lead;
      }
      catch ( Exception ex )
      {
         this.addMessage( ApexPages.severity.ERROR, 'Error saving lead: ' + ex.getMessage() );
      }
      PageReference student = ApexPages.CurrentPage();
      return student;
   }

   public pageReference Cancel ()
   {
      System.Debug('Cancel with >'+this.lead.Last_Viewed__c + '< selected tab >' + this.selectedTab + '<');

      update this.lead;

      PageReference student = ApexPages.CurrentPage();
      return student;
   }
   
   public PageReference removeLoan()
   {
        /* TODO bt - obsolete ?
      update this.Loans;
      this.recalcLoanAmounts();
      */
      this.addMessage( ApexPages.severity.WARNING, 'No more removeLoan needed ? ');      
      return null;
   }

   private void addMessage( ApexPages.Severity lvl, String msg )
   {
      ApexPages.addmessage(new ApexPages.message(lvl , msg ));
   }

   /*
   *  Saves the tab that was last clicked
   */
   public PageReference setLastTab() 
   {
      if (this.lead != null && this.lead.Id != null) 
      {
         this.lead.Last_Viewed__c = Apexpages.currentPage().getParameters().get('tabName');
         //update this.lead;
         this.selectedTab = this.lead.Last_Viewed__c;
      }

      return null;
   }


   public PageReference updateMarriage()
   {
      if ( this.lead.Marital_Status__c == 'Married' ||
         this.lead.Marital_Status__c == 'Separated' )
      {
         this.isMarried = true;
      }

      return null;
   }

   public PageReference updateTaxes()
   {
      if ( this.lead.Taxes_Filed__c == 'Separately' ) 
      {
         this.isSeparateTax = true;
      }

      return null;
   }
   
   

   public Component.Apex.Detail getDisplayDebt() 
   {
      Component.Apex.Detail qd = new Component.Apex.Detail();

      qd.expressions.subject = '{!selectedDebt}';
      qd.relatedList = false;
      qd.title = false;
      qd.inlineEdit = true;

      return qd;
   }


   public pageReference selectQuote()
   {
      List<New_Quote__c> accepted = new List<New_Quote__c>();
      if ( this.selectedQuote != null && this.nquotes.size() > 0 )
      {
         for ( New_Quote__c q : this.nquotes )
         {
            if ( q.Id == this.selectedQuote )
            {
               if ( q.Status__c == 'Accepted' )
               {
                  q.Status__c = 'Pending';
                  this.hasQuote = false;
               }
               else
               {
                  this.hasQuote = true;
                  q.Status__c = 'Accepted';
                  accepted.add(q);
               }
            }
            else if ( q.ParentQuoteID__c == this.selectedQuote )
            {
               if ( q.Status__c == 'Accepted' )
               {
                  q.Status__c = 'Pending';
               }
               else
               {
                  q.Status__c = 'Accepted';
                  accepted.add(q);
               }
            }
         }

         if ( accepted.size() > 0 )
         {
            StudentLoanCalcUtilities.setQuoteAttributes(accepted);
         }
         update this.nquotes;
         wLoadQuotes();
         wBuildQuoteWrappers();
      }
      return null;
   }

   public pageReference saveContact()
   {
      update this.lead;

      Contact refc = new Contact();
            
      if ( this.contact != null && this.contact.Id != null )
      {
         refc = [
            SELECT
            FirstName,
            Id,
            LastName,
            Email,
            MailingCity,
            MailingState,
            MailingCountry,
            MailingPostalCode,
            MailingStreet,
            Phone,
            RecordTypeId,
            Lead__c,
            Relationship__c
            FROM
            Contact
            WHERE ID = :this.contact.Id];
      }

      refc.FirstName = this.contact.FirstName;
      refc.LastName = this.contact.LastName;
      refc.Email = this.contact.Email;
      refc.MailingStreet = this.contact.Street;
      refc.MailingCity = this.contact.City;
      refc.MailingState = this.contact.State;
      refc.MailingCountry = this.contact.Country;
      refc.MailingPostalCode = this.contact.Postal;
      refc.Phone = this.contact.Phone;
      refc.Lead__c = this.lead.Id;
      refc.Relationship__c = this.contact.Relationship;
      refc.RecordTypeId = this.contactRt;

      this.contact = new Reference();

      try {
         upsert refc;
      }
      catch(Exception ex) {
         ApexPages.addMessages(ex);
      }
      this.LoadData();

      return null;
   }

   public pageReference editReference()
   {
      System.Debug('here i am with >');
      Contact c = [
            SELECT
            FirstName,
            Id,
            LastName,
            Email,
            MailingCity,
            MailingState,
            MailingCountry,
            MailingPostalCode,
            MailingStreet,
            Phone,
            RecordTypeId,
            Lead__c,
            Relationship__c
            FROM
            Contact
            WHERE ID = :this.selectedContact];

      this.contact.FirstName = c.FirstName;
      this.contact.LastName = c.LastName;
      this.contact.Email = c.Email;
      this.contact.City = c.MailingCity;
      this.contact.State = c.MailingState;
      this.contact.Country = c.MailingCountry;
      this.contact.Postal = c.MailingPostalCode;
      this.contact.Street = c.MailingStreet;
      this.contact.Phone = c.Phone;
      this.contact.Id = c.Id;
      this.contact.Relationship = c.Relationship__c;

      return null;
   }

   public PageReference deleteContact()
   {
      List<Contact> c = [SELECT ID FROM CONTACT WHERE ID = :this.selectedContact];
      if ( c.size() > 0 )
      {
         delete c;
      }
      this.wLoadContacts();
      return null;
   }

   public PageReference newContact()
   {
      this.selectedContact = null;
      this.contact = new Reference();
      return null;
   }

   public PageReference requote()
   {
      String methodName='requote Recalculate Program';
      System.debug(methodName + ' :: :: '+ ' Entering...' );
      this.wDeleteQuotes();
      
      List<Lead_Debt__c>loansLst =  [SELECT
                      ID,
                      Award_ID__c,
                      Principal_Balance__c,
                      Interest_Balance__c,
                      Interest_Rate__c,
                      Interest_Rate_Type__c,
                      Monthly_Loan_Payment__c,
                      Included__c,
                      Interest_Estimated__c,                    
                      Creditor__r.Name,
                      Loan_Status_Date__c,
                      Loan_Status_Code__c,
                      Loan_Type__c,
                      Verified__c,
                      Loan_Amount__c,
                      Loan_Status__c,
                      Disbursed__c,
                      Loan_Code__c,
                      Lead__c,
                      CanICR__c,
                      CanIBR__c,
                      CanExt__c,
                      CanPayE__c,
                      IsDefault__c,
                      IsClosed__c,
                      RequireIB__c,
                      Loan_Date__c,
                      Loan_Paid__c,
                      Consolidation__c,
                      selected_Paye__c
                      FROM
                      Lead_Debt__c
                      WHERE
                      Lead__r.ID = :this.lead.ID AND selected_Paye__c = true
                      ORDER BY Included__c DESC, Principal_Balance__c DESC, Interest_Balance__c DESC
                    ];
      System.debug(methodName+ ':: loansLst :: '+loansLst);
      
      if(loansLst.size() > 0 && loansLst.get(0).selected_Paye__c==true)
      {
          PDMaps pdm = new PDMaps( loansLst, this.lead ); // creates pdmap objects
      }else
      {
          System.debug(methodName+ ':: loans else :: '+loans);
          PDMaps pdm = new PDMaps( this.loans, this.lead ); // creates pdmap objects
      }
      this.wLoadPDs();
      this.wLoadQuotes();
      
      StudentLoanCalc slpc = new StudentLoanCalc( this.nquotes, this.lead );
      system.debug(':: nquotes :: '+nquotes);
      this.wBuildQuoteWrappers();
      System.debug(methodName+ ':: loansLst :: '+loansLst );
      System.debug(methodName+ ':: Loans :: '+ loans);
     System.debug(methodName + ' :: :: '+ ' Exit...' );
      return null;
   }
    /*public PageReference ConsolidateSumUnSum()
    {
      Boolean flag=false;
      List<New_Quote__c> lstNewQutoe = new List<New_Quote__c>();
      for(Lead_Debt__c ld : this.Loans)
      {
          if(ld.selected_Paye__c==true)
          {
              flag=true;
              for(New_Quote__c nq : this.nquotes)
              {
                  nq.Total_Principal_Paid__c = this.lead.Loan_Amount__c;
                  lstNewQutoe.add(nq);
              }
          }
      }
      if(lstNewQutoe.size() > 0)
      {
          insert lstNewQutoe);
      }
      return null;
    }*/
    
    //List<Lead_Debt__c> selectedLead {get;set;}
    public PageReference processSelected() // this method is not used in any class for VF pages.
    {
    String methodName='processSelected()';
    System.debug(methodName + ' :: :: ' + 'Entering...');
    //selectedLead = new List<Lead_Debt__c>();
        List<Decimal > lstLead= new List<Decimal >();
        Decimal amt ;//=0;// new Decimal ();
        for(Lead_Debt__c leadObj : this.loans) 
        {
            System.debug(methodName + ' :: leadObj  :: ' + leadObj );
            if(leadObj.selected_Paye__c == true) 
            {
                   //this.lead.Loan_Amount__c =(leadObj.Loan_Amount__c + leadObj.Interest_Balance__c); 
                    amt = (leadObj.Loan_Amount__c ); 
                    lstLead.add(amt);           
            }
            
        }
        Decimal total=0;
        if(lstLead.size() > 0)
        {
            for(Decimal d : lstLead)
            {
                total += d;
            }
            this.lead.Loan_Amount__c = total;
            update this.lead;
        }    
        System.debug(methodName + ' :: lead :: ' + lead +':: lstLead :: ' +lstLead);
        System.debug(methodName + ' :: :: ' + ' Exit...');
        return null;
    }

    public PageReference doConsolidateLoansSelected()
   {
      String methodName='doConsolidateLoansSelected()';
      System.debug(methodName + ' :: :: ' + 'Entering...');
      List<String> consolidateLoans = new List<String>();
      
      
      List<Lead_Debt__c>loansLst =  [SELECT
                      ID,
                      Award_ID__c,
                      Principal_Balance__c,
                      Interest_Balance__c,
                      Interest_Rate__c,
                      Interest_Rate_Type__c,
                      Monthly_Loan_Payment__c,
                      Included__c,
                      Interest_Estimated__c,                    
                      Creditor__r.Name,
                      Loan_Status_Date__c,
                      Loan_Status_Code__c,
                      Loan_Type__c,
                      Verified__c,
                      Loan_Amount__c,
                      Loan_Status__c,
                      Disbursed__c,
                      Loan_Code__c,
                      Lead__c,
                      CanICR__c,
                      CanIBR__c,
                      CanExt__c,
                      CanPayE__c,
                      IsDefault__c,
                      IsClosed__c,
                      RequireIB__c,
                      Loan_Date__c,
                      Loan_Paid__c,
                      Consolidation__c,
                      selected_Paye__c
                      FROM
                      Lead_Debt__c
                      WHERE
                      Lead__r.ID = :this.lead.ID AND selected_Paye__c = true
                      ORDER BY Included__c DESC, Principal_Balance__c DESC, Interest_Balance__c DESC
                    ];
     
         
         if(loansLst.size() > 0)
         {
             for ( Lead_Debt__c ld : loansLst)
             {
                if ( ld.IsClosed__c == 'Open Consolidation' && ld.selected_Paye__c == true )
                {
                   consolidateLoans.add(ld.Id);
                }
            }         
         }
     
         System.debug(methodName + ' :: consolidateLoans :: ' + consolidateLoans);
      if (loansLst.size() > 0 &&  this.selectedAction == 'Consolidate' )
      {
         StudentLoanCalcUtilities.consolidate(loansLst, consolidateLoans);
         System.debug(methodName + ' :: consolidateLoans Consolidate :: ' + consolidateLoans+ ':: loansLst :: '+loansLst);
         this.wDeleteQuotes();
         List<Loan_Status_Codes__c> lsc = [
            SELECT
            Status_Code__c,
            IsClosed__c
            FROM Loan_Status_Codes__c
         ];
         StudentLoanCalcUtilities.calcWeighted(this.lead, loansLst, lsc);
         this.wBuildQuoteWrappers();
         update this.lead;
         this.selectedAction = null;
      }
    System.debug(methodName + ' :: :: ' + ' Exit...');
      return null;
   }

   public PageReference doAction()
   {
      String methodName='doAction()';
      System.debug(methodName + ' :: :: ' + 'Entering...');
      List<String> consolidateLoans = new List<String>();
      if ( this.selectedLoans != null )
      {
         consolidateLoans = this.selectedLoans.split(',');
         System.debug(methodName + ' :: consolidateLoans :: ' + consolidateLoans);
         this.selectedLoans = null;
      }
      else
      {
         for ( Lead_Debt__c ld : this.loans )
         {
            if ( ld.IsClosed__c == 'Open' )
            {
               consolidateLoans.add(ld.Id);
            }
         }
      }

      if ( this.selectedAction == 'Consolidate' )
      {
         StudentLoanCalcUtilities.consolidate(this.loans, consolidateLoans);
         System.debug(methodName + ' :: consolidateLoans Consolidate :: ' + consolidateLoans+ ':: loansloans:: '+loans);
         this.wDeleteQuotes();
         List<Loan_Status_Codes__c> lsc = [
            SELECT
            Status_Code__c,
            IsClosed__c
            FROM Loan_Status_Codes__c
         ];
         StudentLoanCalcUtilities.calcWeighted(this.lead, this.loans, lsc);
         this.wBuildQuoteWrappers();
         //update this.lead;// i am comment this update lead b/c below method doing same thing.
         doConsolidateLoansSelected();
         this.selectedAction = null;
      }
    System.debug(methodName + ' :: :: ' + ' Exit...');
      return null;
   }
   public PageReference GenerateScheduleLead()
   {
      if ( StudentPaymentSchedule.GenerateScheduleLead(this.lead) == true )
      {
         this.lead.HasPaymentSchedule__c = true;
         update this.lead;
      }
      return ApexPages.currentPage();
      //this.client.Last_Viewed__c = Apexpages.currentPage().getParameters().get('tabName');
   } 
   
    public StudentLeadController(){
     this.lead =   [SELECT
                     Id,
                     SSN_ENC__c,
                     Last_Name__c,
                     DOB__c,
                     FAFSA_PIN_ENC__c
                     FROM
                     Lead__c
                     WHERE Id = :ApexPages.currentPage().getParameters().get('id')];    
    }
    
    
    public void myCallout(){
        
       this.nsldsPrivacyAcceptNS();
        
    }
  private String nsldsPrivacyAcceptNS()
   {
     String methodName='nsldsPrivacyAcceptNS';//added on 23/09/2014
     System.debug(methodName + ' :: :: ' + ' Entering... ');
     try 
     {
       HttpRequest req = new HttpRequest();
       req.setEndPoint( 'https://www.nslds.ed.gov/nslds_SA/secure/SaFinPrivacyAccept.do' );
       req.setMethod( 'GET' );
       req.setTimeout(120000);
       //req.setHeader('User-Agent','Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/29.0.1547.65 Safari/537.36');
       Http http = new Http();
       HttpResponse res = http.send( req );
       System.debug(methodName + ' :: req  :: ' + req);
       Integer reqlimit = 0;

       System.Debug(methodName +'First shot have >'+res.getHeader('Set-Cookie')+'<');
       System.debug(methodName + ' :: :: ' + ' Exit... ');
      // Cookie cook=new Cookie('JSESSIONID', res.getHeader('Set-Cookie'), null, 1440, false);
       //System.debug(methodName +':: cook.getValue():: '+cook.getValue());
       nsldsLogin( res.getHeader('Set-Cookie'));
       return res.getHeader('Set-Cookie');
     }
     catch (Exception ex)
     {
       //this.retrieveErrors.add('The NSLDS site access failed, the site returned -- ' + ex.getMessage() );
       return null;
     }
     
   }
   private String nsldsLogin( String cookie )
   {
      String methodName='nsldsLogin';//added on 23/09/2014
      System.debug(methodName + ' :: :: ' + ' Entering... ');
       System.debug(methodName + ' :: cookie  :: ' + cookie );
      //  System.Debug('here in nslds login');
      HttpRequest req = new HttpRequest();
      req.setEndpoint( 'https://www.nslds.ed.gov/nslds_SA/secure/SaFinLoginPage.do' );
      req.setMethod( 'GET' );
      //req.setHeader('Content-Length','1024');
      //req.setHeader('Connection','keep-alive');
      if( cookie != null )
      {
         req.setHeader( 'Cookie', cookie );
      }
      req.setTimeout(120000);
      //req.setHeader('User-Agent','Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/29.0.1547.65 Safari/537.36');

      System.Debug('cookie is >'+cookie+'<');

      Http http = new Http();
      //req.setHeader('Content-Type', 'application/x-www-form-urlencoded');
      //req.setHeader('Connection', 'keep-alive');
      HttpResponse res = http.send( req );
      System.debug(methodName + ' :: HttpResponse :: ' +res);
      System.debug(methodName + ' :: HttpResponse body:: ' +res.getBody()+ ' :: getHeader:: '+res.getHeader('Set-Cookie'));
      if ( cookie != null )
      {
         //cookie = res.getHeader('Set-Cookie');
         req.setHeader( 'Cookie', cookie );
      }

      System.Debug('cookie is >'+cookie+'<');

      String body = res.getBody().replaceAll('\r','').replaceAll('\n','');
      body = body.replaceAll('^.*?<form','');
      body = body.replaceAll('</form>.*$','');
      system.debug('::::'+body);
      // get required information from body
      Map<Integer, Integer> pin1 = new Map<Integer, Integer>();
      Map<Integer, Integer> pin2 = new Map<Integer, Integer>();
      Map<Integer, Integer> pin3 = new Map<Integer, Integer>();
      Map<Integer, Integer> pin4 = new Map<Integer, Integer>();
      String sessionId = null;
      String formAction = null;

      Pattern actionPattern = Pattern.compile('.*?action\\s*=\\s*"(.*?)".*');
      Matcher actionMatch = actionPattern.matcher(body);
      if ( actionMatch.matches() )
      {
         formAction = actionMatch.group(1);
      }
      System.debug(methodName+ ' :: formAction :: '+formAction);
      Pattern SessionPattern = Pattern.compile('.*?name\\s*=\\s*"sessionId"\\s*value\\s*=\\s*"(.*?)".*');
      Matcher SessionMatch = SessionPattern.matcher(body);
      if(SessionMatch.matches())
      {
         sessionId = SessionMatch.group(1);
      }
      System.debug(methodName+' :: sessionId  :: '+sessionId );
      pin1 = this.pinMapNS('yin1',body);
      pin2 = this.pinMapNS('yin2',body);
      pin3 = this.pinMapNS('yin3',body);
      pin4 = this.pinMapNS('yin4',body);
      System.debug(methodName+ ' :: body :: '+body);
      String social = (String)this.lead.get('SSN_ENC__c');
      System.debug(methodName+ ' :: social :: '+social );
      String lname = ((String)this.lead.get('Last_Name__c')).substring(0,2).toLowerCase();
      System.debug(methodName+ ' :: lname :: '+lname );
      Date dobd = ( this.lead.getSObjectType() == Lead__c.sObjectType ? (Date)this.lead.get('DOB__c') : (Date)this.lead.get('Date_of_Birth__c') );
      System.debug(methodName+ ' :: dobd :: '+dobd );
      Integer Month = dobd.month();
      Integer Day = dobd.day();
      Integer Year = dobd.Year();
      String dob = ( Month < 10 ? '0' + String.valueOf(Month) : String.valueOf(Month) ) +
         ( Day < 10 ? '0' + String.valueOf(Day) : String.valueOf(Day) ) +
         String.valueOf(Year);
      String pin = ((String)this.lead.get('FAFSA_PIN_ENC__c')).trim();
      System.Debug('PIN >'+pin+'<');
      System.Debug('SSN >'+social+'<');
      String yin1h = String.valueOf(pin1.get(Integer.valueOf(pin.mid(0,1))));
      String yin2h = String.valueOf(pin2.get(Integer.valueOf(pin.mid(1,1))));
      String yin3h = String.valueOf(pin3.get(Integer.valueOf(pin.mid(2,1))));
      String yin4h = String.valueOf(pin4.get(Integer.valueOf(pin.mid(3,1))));

      HttpRequest req2 = new HttpRequest();
      req2.setEndpoint('https://www.nslds.ed.gov' + formAction );
      
      req2.setMethod('POST');
      if ( cookie != null ) 
      {
         req2.setHeader('Cookie', cookie);
      }
      System.debug(methodName+ ' :: 2 cookie:: '+cookie);
      req2.setTimeout(120000);
      //req2.setHeader('User-Agent','Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/29.0.1547.65 Safari/537.36');

      req2.setBody('borrowerSsn='+EncodingUtil.urlEncode(social,'UTF-8') +
                  '&borrowerName='+EncodingUtil.urlEncode(lname,'UTF-8') +
                  '&borrowerDob='+EncodingUtil.urlEncode(dob,'UTF-8') +
                  '&sessionId='+EncodingUtil.urlEncode(sessionId,'UTF-8') +
                  '&yin1h='+EncodingUtil.urlEncode(yin1h,'UTF-8') +
                  '&yin2h='+EncodingUtil.urlEncode(yin2h,'UTF-8') +
                  '&yin3h='+EncodingUtil.urlEncode(yin3h,'UTF-8') +
                  '&yin4h='+EncodingUtil.urlEncode(yin4h,'UTF-8'));
      system.debug('borrowerSsn='+EncodingUtil.urlEncode(social,'UTF-8') +
                  '&borrowerName='+EncodingUtil.urlEncode(lname,'UTF-8') +
                  '&borrowerDob='+EncodingUtil.urlEncode(dob,'UTF-8') +
                  '&sessionId='+EncodingUtil.urlEncode(sessionId,'UTF-8') +
                  '&yin1h='+EncodingUtil.urlEncode(yin1h,'UTF-8') +
                  '&yin2h='+EncodingUtil.urlEncode(yin2h,'UTF-8') +
                  '&yin3h='+EncodingUtil.urlEncode(yin3h,'UTF-8') +
                  '&yin4h='+EncodingUtil.urlEncode(yin4h,'UTF-8'));
      System.debug(methodName+ ' ::req2  OO :: '+req2);
      Http http2 = new Http();
      HttpResponse res2 = http2.send(req2);
      System.debug(methodName+ ' :: 2 res2 :: '+res2 );
      System.debug(methodName+ ' :: 2 res2 body :: '+res2.getBody()+ ':: status :: '+res2.getStatusCode()+' :: :: '+res.getHeader('Set-Cookie'));
      if ( res2.getStatusCode() != 302 )
      {
         cookie = null;
         //this.retrieveErrors.add('NSLDS Login Information is Incorrect.');
         System.debug(methodName+ ' :: 2 res2.getStatusCode():: '+res2.getStatusCode());
      }
      system.debug(':::Res2 Location:::'+res2.getHeader('Location'));
      if ( res2.getHeader('Set-Cookie') != null )
      {
          cookie = res2.getHeader('Set-Cookie');
          req.setHeader( 'Cookie', cookie );
      }
      System.debug(methodName+ ' :: 2 res2 cookie :: '+cookie );
      System.debug(methodName + ' :: :: ' + ' Exit... ');
      req = new HttpRequest();
      req.setEndpoint('https://www.nslds.ed.gov/nslds_SA/secure/MyData/MyStudentData.do?language=en');
      req.setCompressed(true);
      //Cookie cook=new Cookie('JSESSIONID', cookie , null, 1440, false);
      //System.debug(methodName +':: cook.getValue()1111:: '+cook.getValue());
      //req.setHeader('Content-Type', 'text');
      //req.setHeader('Connection', 'keep-alive');
      
      req.setMethod('GET');
      if ( cookie  != null )
      {
          req.setHeader('Cookie',cookie);
          //req.setHeader('Authorization', 'OAuth '+UserInfo.getSessionId());  
      }
      req.setTimeout(120000);
      //req.setHeader('User-Agent','Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/29.0.1547.65 Safari/537.36');
      //req.setCompressed(true); 
      System.debug(methodName + ' ::  cookie  req:: ' + req);
      System.debug(methodName + ' ::  cookie  req Body:: ' + req.getBody());

      Http http3 = new Http();
      HttpResponse res3 = http3.send(req);
      System.debug(methodName + ' ::  cookie  res3 :: ' + res3.getBody() );
      System.debug(methodName + ' ::  res3.toString  :: '+res3.toString());  
      
      for( String x : res3.getHeaderKeys() )
      {
         System.Debug(methodName + 'dl >'+x+'<');
         if( x != null ) 
         {
            System.Debug(methodName + 'with >'+res3.getHeader(x)+'<');
         }
      } 

      System.Debug(methodName + 'SENT THE THING SHOULD GET THE STUFF');

      Blob resAsBlob = res3.getBodyAsBlob();
      System.debug(methodName + ' ::  cookie  resAsBlob :: ' + resAsBlob.toString() );   
      Attachment DataFile = new Attachment();
      DataFile.Name = 'LoanData-'+Datetime.Now()+'.txt';
      DataFile.ParentId = (String)this.lead.get('Id');
      DataFile.body = resAsBlob;

      insert DataFile; 
      return cookie;
   }
   private Map<Integer, Integer> pinMapNS ( String search, String source )
   {
      Map<Integer, Integer> pins = new Map<Integer, Integer>();
      Pattern rawXmlPattern = Pattern.compile('.*?<select.*name\\s*=\\s*"'+search+'".*?>(.*?)</select>.*');
      Matcher rawXmlMatch = rawXmlPattern.matcher(source);

      if ( rawXmlMatch.matches() )
      {
         XmlStreamReader reader = new XmlStreamReader('<select>'+rawXmlMatch.group(1)+'</select>');
         Integer key = null;
         Integer value = null;
         while ( reader.hasNext() )
         {
            if ( reader.getEventType() == XmlTag.START_ELEMENT &&
               reader.getLocalName() == 'option' )
            {
               value = Integer.valueOf(reader.getAttributeValue(null,'value'));
            }
            else if ( reader.getEventType() == XmlTag.CHARACTERS )
            {
               key = Integer.valueOf(reader.getText());
            }
            else if ( reader.getEventType() == XmlTag.END_ELEMENT &&
                     reader.getLocalName() == 'option' )
            {
               pins.put(key,value);
            }
            reader.next();
         }
      }
      return pins;
   }   
}
