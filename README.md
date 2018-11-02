1	Comment: Inicia proceso de tarjeta de crédito
2	Begin Error Handling; Action: Continue; Options: Log to File, Variable Assignment,  Task Status: Fail
3	     Variable Operation: 0 To $APP_STATUS$
4	     Run Task "$AAApplicationPath$\Automation Anywhere\My Tasks\RPACREDCON\TRANSVERSALES\getConfiguration.atmx" @Repeat: Do Not Repeat @Speed: Normal Speed @Pass Variable as argument: $cfgRpaEnabled$; $cfgDbName$; $cfgDbPassword$; $cfgDbServer$; $cfgDbUser$; $cfgMailSmtpHost$; $cfgMailSmtpPort$; $cfgMailUsername$; $MAIN_PATH$; $RPA_USER$
5	     Message Box: "$cfgRpaEnabled$$cfgMailSmtpHost$$cfgMailSmtpPort$$cfgMailUsername$$cfgDbServer$$cfgDbName$$cfgDbUser$$cfgDbPassword$"
6	     Variable Operation: 0 To $vArrayCount$
7	     Comment: New conection used to get execution apps by product.
8	     Begin Error Handling; Action: Continue; Options: Log to File, Variable Assignment,  Task Status: Fail
9	          Connect to "Provider=SQLOLEDB.1;Password=$cfgDbPassword$;Persist Security Info=True;User ID=$cfgDbUser$;Initial Catalog=$cfgDbName$;Data Source=$cfgDbServer$" Session:'dbFeaturesSession'
10	          Run Stored Procedure: '[db_datareader].[up_ProductVariante_SELECT]';  Parameters: '$IN_ID_CASE$';  within 60 seconds;  Session: 'dbFeaturesSession'
11	     End Error Handling
12	     Disconnect from database Session:'dbFeaturesSession'
13	     Start Loop "Each row in SQL Data Set Session: dbFeaturesSession"
14	          Message Box: "ExtraARQA: $Dataset Column(ExtraARQA)$"
15	          Variable Operation: $Dataset Column(ExtraARQA)$ To $vIsFeatureR1Enabled$
16	          Variable Operation: $Dataset Column(ExtraARIB_R2)$ To $vIsFeatureR2Enabled$
17	     End Loop
18	     Message Box: "IsFeatureR2Enabled: [$vIsFeatureR2Enabled$]STOP!"
19	     If $vIsFeatureR1Enabled$ Equal To (=) "1" Then  
20	          Connect to "Provider=SQLOLEDB.1;Password=$cfgDbPassword$;Persist Security Info=True;User ID=$cfgDbUser$;Initial Catalog=$cfgDbName$;Data Source=$cfgDbServer$" Session:'arqaSession'
21	          Begin Error Handling; Action: Continue; Options: Log to File, Variable Assignment,  Task Status: Fail
22	               Message Box: "Parameters: $IN_ID_CASE$$vIndCreditCard$$vIndBankAccount$$vIndCredit$$vIndTargetStbc$$vOutRowCountDS$"
23	               Run Stored Procedure: '[db_datareader].[up_ExtraSgcCoc_SELECT_ProductNumber]';  Parameters: '$IN_ID_CASE$, $vIndCreditCard$, $vIndBankAccount$, $vIndCredit$, $vIndTargetStbc$, $vOutRowCountDS$';  within 60 seconds;  Session: 'arqaSession'
24	          End Error Handling
25	          Message Box: "Registers : $vOutRowCountDS$"
26	          Disconnect from database Session:'arqaSession'
27	          Begin Error Handling; Action: Stop Task; Options: Log to File, Variable Assignment,  Task Status: Fail
28	               If $vOutRowCountDS$ Greater than(>) "0" Then  
29	                    If $isCertEnvironment$ Equal To (=) "true" Then  
30	                         Terminal Emulator : Connect to Terminal "192.168.254.21"; Type: "TN3270"; Session:"Session1"
31	                    Else
32	                         Terminal Emulator : Connect to Terminal "130.1.20.180"; Type: "TN3270"; Session:"Session1"
33	                    End If
34	                    Delay: (1000 ms)
35	                    Terminal Emulator:  Send Text "$RPA_USER$" followed by "KEY_ENTER" to Terminal; Session : "Session1"
36	                    Delay: (1000 ms)
37	                    Terminal Emulator:  Send Text "**************" followed by "KEY_ENTER" to Terminal; Session : "Session1"
38	                    If $isCertEnvironment$ Equal To (=) "true" Then  
39	                         Terminal Emulator:  Send Text "$RPACRECON-Certi(AEXT02)$" followed by "KEY_ENTER" to Terminal; Session : "Session1"
40	                    Else
41	                         If $RPA_USER$ Equal To (=) "RP04P1" Then  
42	                              Comment: Please enter the conditional commands here.
43	                              Terminal Emulator:  Send Text "$RPACRECON(AEXT01)$" followed by "KEY_ENTER" to Terminal; Session : "Session1"
44	                         Else If $RPA_USER$ Equal To (=) "RP04P2" Then  
45	                              Terminal Emulator:  Send Text "$RPACRECON(AEXT02)$" followed by "KEY_ENTER" to Terminal; Session : "Session1"
46	                         Else If $RPA_USER$ Equal To (=) "RP04P3" Then  
47	                              Terminal Emulator:  Send Text "$RPACRECON(AEXT03)$" followed by "KEY_ENTER" to Terminal; Session : "Session1"
48	                         Else If $RPA_USER$ Equal To (=) "RP04P4" Then  
49	                              Terminal Emulator:  Send Text "$RPACRECON(AEXT04)$" followed by "KEY_ENTER" to Terminal; Session : "Session1"
50	                         Else If $RPA_USER$ Equal To (=) "RP04P5" Then  
51	                              Terminal Emulator:  Send Text "$RPACRECON(AEXT05)$" followed by "KEY_ENTER" to Terminal; Session : "Session1"
52	                         End If
53	                    End If
54	                    Delay: (1000 ms)
55	                    Terminal Emulator : Get text from Terminal to "$textCurrentWindow$"; Session:"Session1"
56	                    Message Box: "Connect emulator - terminal "
57	                    If $textCurrentWindow$ Includes $ERROR_LOGIN_BAD_PWD$ OR $textCurrentWindow$ Includes $ERROR_LOGIN_BAD_USER$ Then
58	                         Message Box: "ERROR IN USER LOGIN"
59	                         Run Task "$AAApplicationPath$\Automation Anywhere\My Tasks\RPACREDCON\TRANSVERSALES\CustomExceptionHandler.atmx" @Repeat: Do Not Repeat @Speed: Normal Speed @Pass Variable as argument: $APP_NAME$
60	                         Terminal Emulator : Disconnect from Terminal; Session: "Session1"
61	                    End If
62	               End If
63	               Comment: Abrir Extra sólo cuando existen registros en COC
64	               If $vOutRowCountDS$ Greater than(>) "0" Then  
65	                    If $APP_STATUS$ Equal To (=) "0" Then  
66	                         Start Loop "Each row in SQL Data Set Session: arqaSession"
67	                              Message Box: "DS Counter: [$Counter$]"
68	                              Variable Operation: $Dataset Column(IsVersion85VP)$ To $vIsVersion85VP$
69	                              Message Box: "IsVersion85VP: [$vIsVersion85VP$]"
70	                              Terminal Emulator : Get text from Terminal to "$textCurrentWindow$"; Session:"Session1"
71	                              Comment: Usar VP v2.5 (Todos los casos excepto AMEX y VISA Infinitive)
72	                              If $vIsVersion85VP$ Equal To (=) "0" Then  
73	                                   If $vIsInsideSistemaVP$ Equal To (=) "false" Then  
74	                                        String Operation: Find "$nombreSistema$" within "$textCurrentWindow$" and assign output to $indexNombreSistema$
75	                                        Delay: (500 ms)
76	                                        Variable Operation: $indexNombreSistema$ - 10 To $indexIdNombreSistema$
77	                                        String Operation: Extract substring from "$textCurrentWindow$" and assign output to $idNombreSistema$
78	                                        String Operation: Trim "$idNombreSistema$" and assign output to $idNombreSistema$
79	                                        Message Box: "Id Nombre Sistema: [$idNombreSistema$]"
80	                                        Terminal Emulator:  Send Text "LF $idNombreSistema$" followed by "KEY_ENTER" to Terminal; Session : "Session1"
81	                                        Terminal Emulator:  Send Text "$idNombreSistema$" followed by "KEY_ENTER" to Terminal; Session : "Session1"
82	                                        Variable Operation: true To $vIsInsideSistemaVP$
83	                                        Delay: (1000 ms)
84	                                        Terminal Emulator : Get text from Terminal to "$textCurrentWindow$"; Session:"Session1"
85	                                        Message Box: "$textCurrentWindow$"
86	                                        If $isCertEnvironment$ Equal To (=) "true" Then  
87	                                             Comment: ONLY FOR CERT ENVIRONMENT
88	                                             If $textCurrentWindow$ Includes "EMS1166E" Then  
89	                                                  Message Box: "Error in Extra application - Request Failed for application $nombreSistema$"
90	                                                  Terminal Emulator : Disconnect from Terminal; Session: "Session1"
91	                                                  Stop The Current Task
92	                                             End If
93	                                        End If
94	                                        If $exist2LoginScreen$ Equal To (=) "true" Then  
95	                                             Comment: Please enter the conditional commands here.
96	                                             Terminal Emulator:  Send Text "$RPA_USER$" followed by "KEY_ENTER" to Terminal; Session : "Session1"
97	                                             Delay: (1000 ms)
98	                                             If $isCertEnvironment$ Equal To (=) "true" Then  
99	                                                  Terminal Emulator:  Send Text "$RPACRECON-Certi(AEXT02)$" followed by "KEY_ENTER" to Terminal; Session : "Session1"
100	                                             Else
101	                                                  Terminal Emulator:  Send Text "$RPACRECON(AEXT02)$" followed by "KEY_ENTER" to Terminal; Session : "Session1"
102	                                                  If $RPA_USER$ Equal To (=) "RP04P1" Then  
103	                                                       Comment: Please enter the conditional commands here.
104	                                                       Terminal Emulator:  Send Text "$RPACRECON(AEXT01)$" followed by "KEY_ENTER" to Terminal; Session : "Session1"
105	                                                  Else If $RPA_USER$ Equal To (=) "RP04P2" Then  
106	                                                       Terminal Emulator:  Send Text "$RPACRECON(AEXT02)$" followed by "KEY_ENTER" to Terminal; Session : "Session1"
107	                                                  Else If $RPA_USER$ Equal To (=) "RP04P3" Then  
108	                                                       Terminal Emulator:  Send Text "$RPACRECON(AEXT03)$" followed by "KEY_ENTER" to Terminal; Session : "Session1"
109	                                                  Else If $RPA_USER$ Equal To (=) "RP04P4" Then  
110	                                                       Terminal Emulator:  Send Text "$RPACRECON(AEXT04)$" followed by "KEY_ENTER" to Terminal; Session : "Session1"
111	                                                  Else If $RPA_USER$ Equal To (=) "RP04P5" Then  
112	                                                       Terminal Emulator:  Send Text "$RPACRECON(AEXT05)$" followed by "KEY_ENTER" to Terminal; Session : "Session1"
113	                                                  End If
114	                                             End If
115	                                             Delay: (1000 ms)
116	                                             Terminal Emulator:  Send Text "ARQA" followed by "KEY_ENTER" to Terminal; Session : "Session1"
117	                                             Delay: (1000 ms)
118	                                        End If
119	                                   End If
120	                                   Terminal Emulator : Get text from Terminal to "$textCurrentWindow$"; Session:"Session1"
121	                                   Message Box: "$textCurrentWindow$"
122	                                   If $textCurrentWindow$ Does Not Include "BEHAVIOR HISTORY INQUIRY" Then  
123	                                        Message Box: "Error in Extra application"
124	                                        Variable Operation: 1 To $APP_STATUS$
125	                                        Variable Operation: La opción BEHAVIOR HISTORY INQUIRY No se cargó correctamente To $APP_ERROR_DESCRIPCION$
126	                                        Terminal Emulator : Disconnect from Terminal; Session: "Session1"
127	                                        Stop The Current Task
128	                                   End If
129	                                   Message Box: "$Dataset Column(ProductNumber)$ - $Dataset Column(ProductNumberFormated)$"
130	                                   Variable Operation: $Dataset Column(ProductNumber)$ To $vProductNumber$
131	                                   Variable Operation: 000$vProductNumber$ To $vCreditCardNumber$
132	                                   String Operation: Get length of "$vProductNumber$" and assign output to $vCreditCardLen$
133	                                   If $vCreditCardLen$ Equal To (=) "15" Then  
134	                                        Variable Operation: 0000$vProductNumber$ To $vCreditCardNumber$
135	                                   Else If $vCreditCardLen$ Equal To (=) "16" Then  
136	                                        Variable Operation: 000$vProductNumber$ To $vCreditCardNumber$
137	                                   Else
138	                                        Variable Operation: $vProductNumber$ To $vCreditCardNumber$
139	                                        Message Box: "Credit Card Len - ERROR"
140	                                   End If
141	                                   Message Box: "CreditCardNumber: [$vCreditCardNumber$]"
142	                                   Delay: (500 ms)
143	                                   Terminal Emulator:  Send Text "$vCreditCardNumber$" to Terminal; Session : "Session1"
144	                                   Delay: (500 ms)
145	                                   Variable Operation: $cteDOLARES25$ To $vCurrency$
146	                                   Terminal Emulator:  Send Text "" followed by "KEY_BACKTAB" to Terminal; Session : "Session1"
147	                                   Delay: (500 ms)
148	                                   Start Loop "2" Times 
149	                                        Message Box: "Conter: [$Counter$]"
150	                                        Terminal Emulator:  Send Text "" followed by "KEY_BACKTAB" to Terminal; Session : "Session1"
151	                                        Delay: (500 ms)
152	                                        Terminal Emulator:  Send Text "$vCurrency$" followed by "KEY_ENTER" to Terminal; Session : "Session1"
153	                                        Delay: (1000 ms)
154	                                        Terminal Emulator : Get text from Terminal to "$textCurrentWindow$"; Session:"Session1"
155	                                        If $textCurrentWindow$ Includes "ACCOUNT NOT ON FILE" Then
156	                                             Variable Operation: $cteSOLES25$ To $vCurrency$
157	                                             Delay: (500 ms)
158	                                             Continue
159	                                        Else
160	                                             Delay: (500 ms)
161	                                             Variable Operation: 3 To $numberOfMonths$
162	                                             Message Box: "VP 85$textCurrentWindow$"
163	                                             String Operation: Find "STMT DATE" within "$textCurrentWindow$" and assign output to $vIndexPaymentDateLine$
164	                                             String Operation: Find "TOTAL AMT DUE" within "$textCurrentWindow$" and assign output to $vIndexTotalAmountDueLine$
165	                                             String Operation: Find "AMT PAST DUE" within "$textCurrentWindow$" and assign output to $vIndexAMTPastDue$
166	                                             String Operation: Find "NBR/AMT CSH ADV" within "$textCurrentWindow$" and assign output to $vIndexNBRAMTCshAdv$
167	                                             String Operation: Extract substring from "$textCurrentWindow$" and assign output to $vPaymentDateLine$
168	                                             String Operation: Extract substring from "$textCurrentWindow$" and assign output to $vTotalAmountDueLine$
169	                                             String Operation: Extract substring from "$textCurrentWindow$" and assign output to $vAMTPastDueLine$
170	                                             String Operation: Extract substring from "$textCurrentWindow$" and assign output to $vNBRAMTCshAdvLine$
171	                                             Message Box: "PaymentDateLine: [$vPaymentDateLine$]TotalAmountDueLine: [$vTotalAmountDueLine$]AMTPastDueLine: [$vAMTPastDueLine$]NBRAMTCshAdvLine: [$vNBRAMTCshAdvLine$]"
172	                                             Comment: Obtener datos necesarios y grabar en DB
173	                                             String Operation: Extract substring from "$vPaymentDateLine$" and assign output to $arrPaymentCreditCard(1,1)$
174	                                             String Operation: Extract substring from "$vTotalAmountDueLine$" and assign output to $arrPaymentCreditCard(1,2)$
175	                                             String Operation: Extract substring from "$vAMTPastDueLine$" and assign output to $arrPaymentCreditCard(1,3)$
176	                                             String Operation: Extract substring from "$vNBRAMTCshAdvLine$" and assign output to $arrPaymentCreditCard(1,4)$
177	                                             String Operation: Extract substring from "$vPaymentDateLine$" and assign output to $arrPaymentCreditCard(2,1)$
178	                                             String Operation: Extract substring from "$vTotalAmountDueLine$" and assign output to $arrPaymentCreditCard(2,2)$
179	                                             String Operation: Extract substring from "$vAMTPastDueLine$" and assign output to $arrPaymentCreditCard(2,3)$
180	                                             String Operation: Extract substring from "$vNBRAMTCshAdvLine$" and assign output to $arrPaymentCreditCard(2,4)$
181	                                             String Operation: Extract substring from "$vPaymentDateLine$" and assign output to $arrPaymentCreditCard(3,1)$
182	                                             String Operation: Extract substring from "$vTotalAmountDueLine$" and assign output to $arrPaymentCreditCard(3,2)$
183	                                             String Operation: Extract substring from "$vAMTPastDueLine$" and assign output to $arrPaymentCreditCard(3,3)$
184	                                             String Operation: Extract substring from "$vNBRAMTCshAdvLine$" and assign output to $arrPaymentCreditCard(3,4)$
185	                                             Message Box: "STOP!"
186	                                             If $vIsFeatureR2Enabled$ Equal To (=) "1" Then  
187	                                                  Variable Operation: 6 To $numberOfMonths$
188	                                                  Comment: Go to next page.
189	                                                  Terminal Emulator:  Send Text "" followed by "KEY_PF6" to Terminal; Session : "Session1"
190	                                                  Terminal Emulator : Get text from Terminal to "$textCurrentWindow$"; Session:"Session1"
191	                                                  Message Box: "textCurrentWindow:$textCurrentWindow$"
192	                                                  String Operation: Find "STMT DATE" within "$textCurrentWindow$" and assign output to $vIndexPaymentDateLine$
193	                                                  String Operation: Find "TOTAL AMT DUE" within "$textCurrentWindow$" and assign output to $vIndexTotalAmountDueLine$
194	                                                  String Operation: Find "AMT PAST DUE" within "$textCurrentWindow$" and assign output to $vIndexAMTPastDue$
195	                                                  String Operation: Find "NBR/AMT CSH ADV" within "$textCurrentWindow$" and assign output to $vIndexNBRAMTCshAdv$
196	                                                  String Operation: Extract substring from "$textCurrentWindow$" and assign output to $vPaymentDateLine$
197	                                                  String Operation: Extract substring from "$textCurrentWindow$" and assign output to $vTotalAmountDueLine$
198	                                                  String Operation: Extract substring from "$textCurrentWindow$" and assign output to $vAMTPastDueLine$
199	                                                  String Operation: Extract substring from "$textCurrentWindow$" and assign output to $vNBRAMTCshAdvLine$
200	                                                  Message Box: "PaymentDateLine: [$vPaymentDateLine$]TotalAmountDueLine: [$vTotalAmountDueLine$]AMTPastDueLine: [$vAMTPastDueLine$]NBRAMTCshAdvLine: [$vNBRAMTCshAdvLine$]"
201	                                                  String Operation: Extract substring from "$vPaymentDateLine$" and assign output to $arrPaymentCreditCard(4,1)$
202	                                                  String Operation: Extract substring from "$vTotalAmountDueLine$" and assign output to $arrPaymentCreditCard(4,2)$
203	                                                  String Operation: Extract substring from "$vAMTPastDueLine$" and assign output to $arrPaymentCreditCard(4,3)$
204	                                                  String Operation: Extract substring from "$vNBRAMTCshAdvLine$" and assign output to $arrPaymentCreditCard(4,4)$
205	                                                  String Operation: Extract substring from "$vPaymentDateLine$" and assign output to $arrPaymentCreditCard(5,1)$
206	                                                  String Operation: Extract substring from "$vTotalAmountDueLine$" and assign output to $arrPaymentCreditCard(5,2)$
207	                                                  String Operation: Extract substring from "$vAMTPastDueLine$" and assign output to $arrPaymentCreditCard(5,3)$
208	                                                  String Operation: Extract substring from "$vNBRAMTCshAdvLine$" and assign output to $arrPaymentCreditCard(5,4)$
209	                                                  String Operation: Extract substring from "$vPaymentDateLine$" and assign output to $arrPaymentCreditCard(6,1)$
210	                                                  String Operation: Extract substring from "$vTotalAmountDueLine$" and assign output to $arrPaymentCreditCard(6,2)$
211	                                                  String Operation: Extract substring from "$vAMTPastDueLine$" and assign output to $arrPaymentCreditCard(6,3)$
212	                                                  String Operation: Extract substring from "$vNBRAMTCshAdvLine$" and assign output to $arrPaymentCreditCard(6,4)$
213	                                             End If
214	                                             Comment: Save rocords to database.
215	                                             Variable Operation: 0 To $arrCounter$
216	                                             Start Loop "$numberOfMonths$" Times 
217	                                                  Variable Operation: $arrCounter$+1 To $arrCounter$
218	                                                  Message Box: "DataSetDate: [$arrPaymentCreditCard($arrCounter$,1)$]Amount: [$arrPaymentCreditCard($arrCounter$,2)$]Past Due: [$arrPaymentCreditCard($arrCounter$,3)$]Csh Adv: [$arrPaymentCreditCard($arrCounter$,4)$]"
219	                                                  String Operation: Extract substring from "$arrPaymentCreditCard($arrCounter$,1)$" and assign output to $vYear$
220	                                                  String Operation: Extract substring from "$arrPaymentCreditCard($arrCounter$,1)$" and assign output to $vMonth$
221	                                                  String Operation: Extract substring from "$arrPaymentCreditCard($arrCounter$,1)$" and assign output to $vDay$
222	                                                  Variable Operation: $vYear$$vMonth$$vDay$ To $vPaymentDate$
223	                                                  If $vPaymentDate$ Equal To (=) "00000000" Then  
224	                                                       Variable Operation: 19000101 To $vPaymentDate$
225	                                                  End If
226	                                                  Variable Operation: $arrPaymentCreditCard($arrCounter$,2)$ To $vTotalAmountDue$
227	                                                  Variable Operation: $arrPaymentCreditCard($arrCounter$,3)$ To $vAmtPastDue$
228	                                                  Variable Operation: $arrPaymentCreditCard($arrCounter$,4)$ To $vNbrAmtCshAdv$
229	                                                  String Operation: Replace "," with "" in "$vTotalAmountDue$" and assign output to $vTotalAmountDue$ 
230	                                                  String Operation: Replace "," with "" in "$vAmtPastDue$" and assign output to $vAmtPastDue$ 
231	                                                  String Operation: Replace "," with "" in "$vNbrAmtCshAdv$" and assign output to $vNbrAmtCshAdv$ 
232	                                                  Message Box: "IdCase: [$IN_ID_CASE$]ProductNumber: [$vProductNumber$]Date: [$vPaymentDate$]Amount: [$vTotalAmountDue$]Past Due: [$vAmtPastDue$]NbrAmt Csh Adv: [$vNbrAmtCshAdv$]Currency: [$vCurrency$]"
233	                                                  Connect to "Provider=SQLOLEDB.1;Password=$cfgDbPassword$;Persist Security Info=True;User ID=$cfgDbUser$;Initial Catalog=$cfgDbName$;Data Source=$cfgDbServer$" Session:'extraAribSession'
234	                                                  Begin Error Handling; Action: Continue; Options: Log to File, Variable Assignment,  Task Status: Fail
235	                                                       Run Stored Procedure: 'db_datareader.up_ExtraSgcArib_INSERT_R2';  Parameters: '$IN_ID_CASE$, $vProductNumber$, $vPaymentDate$, $vTotalAmountDue$, $vAmtPastDue$, $vNbrAmtCshAdv$, $vCurrency$, $APP_ERROR_DESCRIPCION$, $APP_STATUS$';  within 60 seconds;  Session: 'extraAribSession'
236	                                                  End Error Handling
237	                                                  Disconnect from database Session:'extraAribSession'
238	                                             End Loop
239	                                             Comment: PAUSA KEY => KEY_CLEAR (AA)
240	                                             Terminal Emulator:  Send Text "" followed by "KEY_CLEAR" to Terminal; Session : "Session1"
241	                                             Message Box: "Se obtuvo la data y se presiona PAUSE."
242	                                        End If
243	                                        Comment: Get data (end)...
244	                                        Message Box: "Obtuvo Valores en [$vCurrency$]"
245	                                        Variable Operation: $cteSOLES25$ To $vCurrency$
246	                                        Delay: (1000 ms)
247	                                   End Loop
248	                              End If
249	                         End Loop
250	                         Message Box: "Paso 2 . 1. 5"
251	                         Delay: (500 ms)
252	                         Terminal Emulator:  Send Text "" followed by "KEY_PA2" to Terminal; Session : "Session1"
253	                         Delay: (1000 ms)
254	                         Terminal Emulator:  Send Text "LF $idNombreSistema$" followed by "KEY_ENTER" to Terminal; Session : "Session1"
255	                         Delay: (1000 ms)
256	                         Variable Operation: false To $vIsInsideSistemaVP$
257	                         Start Loop "Each row in SQL Data Set Session: aribSession"
258	                              Message Box: "Paso 2 . 2"
259	                              Variable Operation: $Dataset Column(IsVersion85VP)$ To $vIsVersion85VP$
260	                              Terminal Emulator : Get text from Terminal to "$textCurrentWindow$"; Session:"Session1"
261	                              Comment: Usar VP v8.5 (Casos AMEX y VISA Infinitive)
262	                              If $vIsVersion85VP$ Equal To (=) "1" Then  
263	                                   If $vIsInsideSistemaVP$ Equal To (=) "false" Then  
264	                                        String Operation: Find "$nombreSistema85$" within "$textCurrentWindow$" and assign output to $indexNombreSistema85$
265	                                        Delay: (500 ms)
266	                                        Variable Operation: $indexNombreSistema85$ - 10 To $indexIdNombreSistema85$
267	                                        String Operation: Extract substring from "$textCurrentWindow$" and assign output to $idNombreSistema85$
268	                                        String Operation: Trim "$idNombreSistema85$" and assign output to $idNombreSistema85$
269	                                        Message Box: "Id Nombre Sistema: [$idNombreSistema$]"
270	                                        Terminal Emulator:  Send Text "LF $idNombreSistema85$" followed by "KEY_ENTER" to Terminal; Session : "Session1"
271	                                        Terminal Emulator:  Send Text "$idNombreSistema85$" followed by "KEY_ENTER" to Terminal; Session : "Session1"
272	                                        Variable Operation: true To $vIsInsideSistemaVP$
273	                                        Delay: (1000 ms)
274	                                        Terminal Emulator : Get text from Terminal to "$textCurrentWindow$"; Session:"Session1"
275	                                        Message Box: "$textCurrentWindow$"
276	                                        If $isCertEnvironment$ Equal To (=) "true" Then  
277	                                             Comment: ONLY FOR CERT ENVIRONMENT
278	                                             If $textCurrentWindow$ Includes "EMS1166E" Then  
279	                                                  Message Box: "Error in Extra application - Request Failed for application $nombreSistema$"
280	                                                  Terminal Emulator : Disconnect from Terminal; Session: "Session1"
281	                                             End If
282	                                        End If
283	                                        If $exist2LoginScreen$ Equal To (=) "true" Then  
284	                                             Comment: Please enter the conditional commands here.
285	                                             Terminal Emulator:  Send Text "$RPA_USER$" followed by "KEY_ENTER" to Terminal; Session : "Session1"
286	                                             Delay: (1000 ms)
287	                                             If $isCertEnvironment$ Equal To (=) "true" Then  
288	                                                  Terminal Emulator:  Send Text "$RPACRECON-Certi(AEXT02)$" followed by "KEY_ENTER" to Terminal; Session : "Session1"
289	                                             Else
290	                                                  Terminal Emulator:  Send Text "$RPACRECON(AEXT02)$" followed by "KEY_ENTER" to Terminal; Session : "Session1"
291	                                             End If
292	                                             Delay: (1000 ms)
293	                                        End If
294	                                        Terminal Emulator:  Send Text "ARQA" followed by "KEY_ENTER" to Terminal; Session : "Session1"
295	                                        Delay: (1000 ms)
296	                                   End If
297	                                   Delay: (1000 ms)
298	                                   Terminal Emulator : Get text from Terminal to "$textCurrentWindow$"; Session:"Session1"
299	                                   Message Box: "$textCurrentWindow$"
300	                                   If $textCurrentWindow$ Does Not Include "BEHAVIOR HISTORY INQUIRY" Then  
301	                                        Message Box: "Error in Extra application"
302	                                        Variable Operation: 1 To $APP_STATUS$
303	                                        Variable Operation: La opción BEHAVIOR HISTORY INQUIRY No se cargó correctamente To $APP_ERROR_DESCRIPCION$
304	                                        Terminal Emulator : Disconnect from Terminal; Session: "Session1"
305	                                        Stop The Current Task
306	                                   End If
307	                                   Message Box: "$Dataset Column(ProductNumber)$ - $Dataset Column(ProductNumberFormated)$"
308	                                   Delay: (500 ms)
309	                                   Variable Operation: $Dataset Column(ProductNumber)$ To $vProductNumber$
310	                                   String Operation: Get length of "$vProductNumber$" and assign output to $vCreditCardLen$
311	                                   If $vCreditCardLen$ Equal To (=) "15" Then  
312	                                        Variable Operation: 0000$vProductNumber$ To $vCreditCardNumber$
313	                                   Else If $vCreditCardLen$ Equal To (=) "16" Then  
314	                                        Variable Operation: 000$vProductNumber$ To $vCreditCardNumber$
315	                                   Else
316	                                        Variable Operation: $vProductNumber$ To $vCreditCardNumber$
317	                                        Message Box: "Credit Card Len - ERROR"
318	                                   End If
319	                                   Variable Operation: 377898050934019 To $vCreditCardNumber$
320	                                   Message Box: "VP 85CreditCardNumber: [$vCreditCardNumber$]"
321	                                   Terminal Emulator:  Send Text "$vCreditCardNumber$" followed by "KEY_ENTER" to Terminal; Session : "Session1"
322	                                   Delay: (500 ms)
323	                                   Terminal Emulator:  Send Text "$vCreditCardNumber$" to Terminal; Session : "Session1"
324	                                   Message Box: "Se envió el número de tarjeta de credito"
325	                                   Delay: (500 ms)
326	                                   Variable Operation: $cteDOLARES85$ To $vCurrency$
327	                                   Terminal Emulator:  Send Text "" followed by "KEY_BACKTAB" to Terminal; Session : "Session1"
328	                                   Delay: (500 ms)
329	                                   Start Loop "2" Times 
330	                                        Message Box: "Conter: [$Counter$]"
331	                                        Terminal Emulator:  Send Text "" followed by "KEY_BACKTAB" to Terminal; Session : "Session1"
332	                                        Delay: (500 ms)
333	                                        Terminal Emulator:  Send Text "$vCurrency$" followed by "KEY_ENTER" to Terminal; Session : "Session1"
334	                                        Message Box: "Getting data..."
335	                                        Delay: (1000 ms)
336	                                        Terminal Emulator : Get text from Terminal to "$textCurrentWindow$"; Session:"Session1"
337	                                        Message Box: "$textCurrentWindow$"
338	                                        Delay: (500 ms)
339	                                        If $textCurrentWindow$ Includes "ACCOUNT NOT ON FILE" Then
340	                                             Variable Operation: $cteSOLES85$ To $vCurrency$
341	                                             Delay: (500 ms)
342	                                             Continue
343	                                             Message Box: "Se mostró pantalla Account Not on File"
344	                                             Comment: TARJETA NO TIENE DATOS
345	                                        Else
346	                                             If $textCurrentWindow$ Includes "TRANSFERRED ACCOUNT" Then
347	                                                  Terminal Emulator:  Send Text "X" followed by "KEY_ENTER" to Terminal; Session : "Session1"
348	                                                  Delay: (500 ms)
349	                                                  Terminal Emulator : Get text from Terminal to "$textCurrentWindow$"; Session:"Session1"
350	                                             End If
351	                                             Delay: (2000 ms)
352	                                             Message Box: "$textCurrentWindow$"
353	                                             String Operation: Find "CYC DATE" within "$textCurrentWindow$" and assign output to $vIndexPaymentDateLine$
354	                                             String Operation: Find "TOTAL AMT DUE" within "$textCurrentWindow$" and assign output to $vIndexTotalAmountDueLine$
355	                                             String Operation: Find "AMT PAST DUE" within "$textCurrentWindow$" and assign output to $vIndexAMTPastDue$
356	                                             String Operation: Find "NUM/AMT CSH ADV" within "$textCurrentWindow$" and assign output to $vIndexNBRAMTCshAdv$
357	                                             Message Box: "IndexPaymentDateLine: [$vIndexPaymentDateLine$]IndexTotalAmountDueLine: [$vIndexTotalAmountDueLine$]IndexAMTPastDueLine: [$vIndexAMTPastDue$]IndexNBRAMTCshAdvLine: [$vIndexNBRAMTCshAdv$]"
358	                                             String Operation: Extract substring from "$textCurrentWindow$" and assign output to $vPaymentDateLine$
359	                                             String Operation: Extract substring from "$textCurrentWindow$" and assign output to $vTotalAmountDueLine$
360	                                             String Operation: Extract substring from "$textCurrentWindow$" and assign output to $vAMTPastDueLine$
361	                                             String Operation: Extract substring from "$textCurrentWindow$" and assign output to $vNBRAMTCshAdvLine$
362	                                             Message Box: "PaymentDateLine: [$vPaymentDateLine$]TotalAmountDueLine: [$vTotalAmountDueLine$]AmtPastDueLine: [$vAMTPastDueLine$]NbrAmtCshAdvLine: [$vNBRAMTCshAdvLine$]"
363	                                             Comment: Obtener datos necesarios y grabar en DB
364	                                             String Operation: Extract substring from "$vPaymentDateLine$" and assign output to $arrPaymentCreditCard(1,1)$
365	                                             String Operation: Extract substring from "$vTotalAmountDueLine$" and assign output to $arrPaymentCreditCard(1,2)$
366	                                             String Operation: Extract substring from "$vAMTPastDueLine$" and assign output to $arrPaymentCreditCard(1,3)$
367	                                             String Operation: Extract substring from "$vNBRAMTCshAdvLine$" and assign output to $arrPaymentCreditCard(1,4)$
368	                                             String Operation: Extract substring from "$vPaymentDateLine$" and assign output to $arrPaymentCreditCard(2,1)$
369	                                             String Operation: Extract substring from "$vTotalAmountDueLine$" and assign output to $arrPaymentCreditCard(2,2)$
370	                                             String Operation: Extract substring from "$vAMTPastDueLine$" and assign output to $arrPaymentCreditCard(2,3)$
371	                                             String Operation: Extract substring from "$vNBRAMTCshAdvLine$" and assign output to $arrPaymentCreditCard(2,4)$
372	                                             Comment: Paginación para estraer del tercer y cuarto mes (anterior)
373	                                             Message Box: "Go to next page..."
374	                                             Terminal Emulator:  Send Text "" followed by "KEY_PF6" to Terminal; Session : "Session1"
375	                                             Delay: (500 ms)
376	                                             Comment: Extraer data del tercer y cuarto mes (anterior)
377	                                             Terminal Emulator : Get text from Terminal to "$textCurrentWindow$"; Session:"Session1"
378	                                             Message Box: "$textCurrentWindow$"
379	                                             String Operation: Find "CYC DATE" within "$textCurrentWindow$" and assign output to $vIndexPaymentDateLine$
380	                                             String Operation: Find "TOTAL AMT DUE" within "$textCurrentWindow$" and assign output to $vIndexTotalAmountDueLine$
381	                                             String Operation: Find "AMT PAST DUE" within "$textCurrentWindow$" and assign output to $vIndexAMTPastDue$
382	                                             String Operation: Find "NUM/AMT CSH ADV" within "$textCurrentWindow$" and assign output to $vIndexNBRAMTCshAdv$
383	                                             String Operation: Extract substring from "$textCurrentWindow$" and assign output to $vPaymentDateLine$
384	                                             String Operation: Extract substring from "$textCurrentWindow$" and assign output to $vTotalAmountDueLine$
385	                                             String Operation: Extract substring from "$textCurrentWindow$" and assign output to $vAMTPastDueLine$
386	                                             String Operation: Extract substring from "$textCurrentWindow$" and assign output to $vNBRAMTCshAdvLine$
387	                                             Message Box: "PaymentDateLine: [$vPaymentDateLine$]TotalAmountDueLine: [$vTotalAmountDueLine$]AmtPastDueLine: [$vAMTPastDueLine$]NbrAmtCshAdvLine: [$vNBRAMTCshAdvLine$]"
388	                                             String Operation: Extract substring from "$vPaymentDateLine$" and assign output to $arrPaymentCreditCard(3,1)$
389	                                             String Operation: Extract substring from "$vTotalAmountDueLine$" and assign output to $arrPaymentCreditCard(3,2)$
390	                                             String Operation: Extract substring from "$vAMTPastDueLine$" and assign output to $arrPaymentCreditCard(3,3)$
391	                                             String Operation: Extract substring from "$vNBRAMTCshAdvLine$" and assign output to $arrPaymentCreditCard(3,4)$
392	                                             String Operation: Extract substring from "$vPaymentDateLine$" and assign output to $arrPaymentCreditCard(4,1)$
393	                                             String Operation: Extract substring from "$vTotalAmountDueLine$" and assign output to $arrPaymentCreditCard(4,2)$
394	                                             String Operation: Extract substring from "$vAMTPastDueLine$" and assign output to $arrPaymentCreditCard(4,3)$
395	                                             String Operation: Extract substring from "$vNBRAMTCshAdvLine$" and assign output to $arrPaymentCreditCard(4,4)$
396	                                             Comment: Extraer data del tercer y cuarto mes (anterior)
397	                                             If $vIsFeatureR2Enabled$ Equal To (=) "1" Then  
398	                                                  Variable Operation: 6 To $numberOfMonths$
399	                                                  Message Box: "Go to next page..."
400	                                                  Comment: Paginación para estraer del tercer y cuarto mes (anterior)
401	                                                  Terminal Emulator:  Send Text "" followed by "KEY_PF6" to Terminal; Session : "Session1"
402	                                                  Delay: (500 ms)
403	                                                  Terminal Emulator : Get text from Terminal to "$textCurrentWindow$"; Session:"Session1"
404	                                                  String Operation: Find "CYC DATE" within "$textCurrentWindow$" and assign output to $vIndexPaymentDateLine$
405	                                                  String Operation: Find "TOTAL AMT DUE" within "$textCurrentWindow$" and assign output to $vIndexTotalAmountDueLine$
406	                                                  String Operation: Find "AMT PAST DUE" within "$textCurrentWindow$" and assign output to $vIndexAMTPastDue$
407	                                                  String Operation: Find "NUM/AMT CSH ADV" within "$textCurrentWindow$" and assign output to $vIndexNBRAMTCshAdv$
408	                                                  String Operation: Extract substring from "$textCurrentWindow$" and assign output to $vPaymentDateLine$
409	                                                  String Operation: Extract substring from "$textCurrentWindow$" and assign output to $vTotalAmountDueLine$
410	                                                  String Operation: Extract substring from "$textCurrentWindow$" and assign output to $vAMTPastDueLine$
411	                                                  String Operation: Extract substring from "$textCurrentWindow$" and assign output to $vNBRAMTCshAdvLine$
412	                                                  Message Box: "PaymentDateLine: [$vPaymentDateLine$]TotalAmountDueLine: [$vTotalAmountDueLine$]AmtPastDueLine: [$vAMTPastDueLine$]NbrAmtCshAdvLine: [$vNBRAMTCshAdvLine$]"
413	                                                  String Operation: Extract substring from "$vPaymentDateLine$" and assign output to $arrPaymentCreditCard(5,1)$
414	                                                  String Operation: Extract substring from "$vTotalAmountDueLine$" and assign output to $arrPaymentCreditCard(5,2)$
415	                                                  String Operation: Extract substring from "$vAMTPastDueLine$" and assign output to $arrPaymentCreditCard(5,3)$
416	                                                  String Operation: Extract substring from "$vNBRAMTCshAdvLine$" and assign output to $arrPaymentCreditCard(5,4)$
417	                                                  String Operation: Extract substring from "$vPaymentDateLine$" and assign output to $arrPaymentCreditCard(6,1)$
418	                                                  String Operation: Extract substring from "$vTotalAmountDueLine$" and assign output to $arrPaymentCreditCard(6,2)$
419	                                                  String Operation: Extract substring from "$vAMTPastDueLine$" and assign output to $arrPaymentCreditCard(6,3)$
420	                                                  String Operation: Extract substring from "$vNBRAMTCshAdvLine$" and assign output to $arrPaymentCreditCard(6,4)$
421	                                             End If
422	                                             Comment: Save records to database.
423	                                             Variable Operation: 0 To $arrCounter$
424	                                             Start Loop "$numberOfMonths$" Times 
425	                                                  Variable Operation: $arrCounter$+1 To $arrCounter$
426	                                                  Message Box: "DataSetDate: [$arrPaymentCreditCard($arrCounter$,1)$]Amount: [$arrPaymentCreditCard($arrCounter$,2)$]"
427	                                                  String Operation: Extract substring from "$arrPaymentCreditCard($arrCounter$,1)$" and assign output to $vYear$
428	                                                  String Operation: Extract substring from "$arrPaymentCreditCard($arrCounter$,1)$" and assign output to $vMonth$
429	                                                  String Operation: Extract substring from "$arrPaymentCreditCard($arrCounter$,1)$" and assign output to $vDay$
430	                                                  Variable Operation: $vYear$$vMonth$$vDay$ To $vPaymentDate$
431	                                                  If $vPaymentDate$ Equal To (=) "00000000" Then  
432	                                                       Variable Operation: 19000101 To $vPaymentDate$
433	                                                  End If
434	                                                  Variable Operation: $arrPaymentCreditCard($arrCounter$,2)$ To $vTotalAmountDue$
435	                                                  Variable Operation: $arrPaymentCreditCard($arrCounter$,3)$ To $vAmtPastDue$
436	                                                  Variable Operation: $arrPaymentCreditCard($arrCounter$,4)$ To $vNbrAmtCshAdv$
437	                                                  String Operation: Replace "," with "" in "$vTotalAmountDue$" and assign output to $vTotalAmountDue$ 
438	                                                  String Operation: Replace "," with "" in "$vAmtPastDue$" and assign output to $vAmtPastDue$ 
439	                                                  String Operation: Replace "," with "" in "$vNbrAmtCshAdv$" and assign output to $vNbrAmtCshAdv$ 
440	                                                  Message Box: "IdCase: [$IN_ID_CASE$]ProductNumber: [$vProductNumber$]Date: [$vPaymentDate$]Amount: [$vTotalAmountDue$]Past Due: [$vAmtPastDue$]NbrAmt Csh Adv: [$vNbrAmtCshAdv$]Currency: [$vCurrency$]"
441	                                                  Connect to "Provider=SQLOLEDB.1;Password=$cfgDbPassword$;Persist Security Info=True;User ID=$cfgDbUser$;Initial Catalog=$cfgDbName$;Data Source=$cfgDbServer$" Session:'extraAribSession'
442	                                                  Begin Error Handling; Action: Continue; Options: Log to File, Variable Assignment,  Task Status: Fail
443	                                                       Run Stored Procedure: 'db_datareader.up_ExtraSgcArib_INSERT_R2';  Parameters: '$IN_ID_CASE$, $vProductNumber$, $vPaymentDate$, $vTotalAmountDue$, $vAmtPastDue$, $vNbrAmtCshAdv$, $vCurrency$, $APP_ERROR_DESCRIPCION$, $APP_STATUS$';  within 60 seconds;  Session: 'extraAribSession'
444	                                                       Run Stored Procedure: 'db_datareader.up_ExtraSgcArib_INSERT';  Parameters: '$IN_ID_CASE$, $vProductNumber$, $vPaymentDate$, $vTotalAmountDue$, $vCurrency$, $APP_ERROR_DESCRIPCION$, $APP_STATUS$';  within 60 seconds;  Session: 'extraAribSession'
445	                                                  End Error Handling
446	                                                  Disconnect from database Session:'extraAribSession'
447	                                             End Loop
448	                                             Message Box: "Se obtuvo la data y se presiona PAUSE."
449	                                             Comment: PAUSA KEY => KEY_CLEAR (AA)
450	                                             Terminal Emulator:  Send Text "" followed by "KEY_CLEAR" to Terminal; Session : "Session1"
451	                                        End If
452	                                        Message Box: "Obtuvo Valores en [$vCurrency$]"
453	                                        Variable Operation: $cteSOLES85$ To $vCurrency$
454	                                        Delay: (1000 ms)
455	                                   End Loop
456	                              End If
457	                         End Loop
458	                         Terminal Emulator:  Send Text "" followed by "KEY_PA2" to Terminal; Session : "Session1"
459	                         Delay: (500 ms)
460	                         Terminal Emulator:  Send Text "LF $idNombreSistema85$" followed by "KEY_ENTER" to Terminal; Session : "Session1"
461	                         Delay: (500 ms)
462	                         Terminal Emulator : Get text from Terminal to "$textCurrentWindow$"; Session:"Session1"
463	                         Delay: (500 ms)
464	                         If $textCurrentWindow$ Includes "You are now logged off" Then  
465	                              Comment: Please enter the conditional commands here.
466	                              Terminal Emulator:  Send Text "LOGOFF" followed by "KEY_ENTER" to Terminal; Session : "Session1"
467	                              Delay: (1000 ms)
468	                              Terminal Emulator : Disconnect from Terminal; Session: "Session1"
469	                         End If
470	                    End If
471	               End If
472	          End Error Handling
473	     End If
474	End Error Handling
475	Message Box: "appStatus: [$APP_STATUS$]"
