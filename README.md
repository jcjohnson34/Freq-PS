# freq

This is a port of Mark Baggett's freq.py into PowerShell! See https://github.com/MarkBaggett/freq for the inspiration.  The Security Onion integration has been so powerful that the goal here is to extend availability of the technique to Blue Teamers doing initial triage and analysis via PowerShell.

Use cases involve analyzing process names, file names, services, etc. 

## Usage

Dot-source freq.ps1 into the current session.  Create a new instance of the FreqCounter class and load a table (see  https://github.com/MarkBaggett/freq/blob/master/freqtable2018.freq for a great starting point).

```PowerShell
PS C:\> . C:\scripts\freq.ps1
PS C:\> $fc = New-Object -TypeName FreqCounter
PS C:\> $fc.LoadHashTable("C:\Scripts\freqtable2018.freq")
```

Once loaded, analyze raw strings, properties, etc. using Get-FrequencyScore:

```PowerShell
PS C:\> Get-FrequencyScore -Measure "abcd"

InputValue FrequencyScore
---------- --------------
abcd       (0.6331,1.243)


PS C:\> Get-ChildItem | Get-FrequencyScore -Property Name |Select-Object -Property Name,FrequencyScore | Sort-Object -Property FrequencyScore -Unique

Name                   FrequencyScore
----                   --------------
hpCqamxUon.exe         (2.2694,2.9836)
```

## Examples

Process Names:

```PowerShell
PS C:\> Get-Process | Get-FrequencyScore -Property Name | Select-Object -Property ProcessName,FrequencyScore -Unique

ProcessName                                                    FrequencyScore
-----------                                                    --------------
ApMsgFwd                                                       (0.7245,0.9421)
ApntEx                                                         (3.9327,4.1761)
Apoint                                                         (10.1262,9.3085)
...
```

Requested DNS Names (Sysmon log source):

```PowerShell
PS C:\> Get-WinEvent -FilterHashtable @{LogName="Microsoft-Windows-Sysmon/Operational";Id=22} |Select-Object -ExpandProperty Message | ForEach-Object {$_.split("`n")[5].split(" ")[1]} | Get-FrequencyScore | Sort-Object -Property FrequencyScore -Unique

InputValue                                     FrequencyScore
----------                                     --------------
fzkjtvb...                                     (0.0129,0.0023)
mfhltwrbmjxf...                                (0.3529,0.4819)
uqcfqrqkmijs...                                (0.6996,0.7087)
bgccbmffzgupnj...                              (1.0023,1.1024)
ppjnnxojifw...                                 (1.0398,0.8954)
eksaczfqxw...                                  (1.2574,1.7907)
oxagnnufkcyudye...                             (1.4576,0.8358)
lnvdczfauzacc...                               (1.6787,1.4209)
...
```

File names:

```PowerShell
PS C:\temp\demo> Get-ChildItem | Get-FrequencyScore -Property Name |Select-Object -Property Name,FrequencyScore | Sort-Object -Property FrequencyScore -Unique

Name                   FrequencyScore
----                   --------------
hpCqamxUon.exe         (2.2694,2.9836)
financials.docx        (6.3278,7.5984)
legitimatefile.txt     (8.1515,6.005)
masterspreadsheet.xlsx (8.6457,8.4249)
```
