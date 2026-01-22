# autocad-msedgewebview2-error

Merhaba, özellikle Autocad cracklerinde ekrana sürekli gelen pencereyi aşağıdaki yöntemler ile birlikte kapatabilirsiniz. İlk iki kod powershell üzerinden msedgewebview2'nin açılmasını engelleyerek çalışıyor. Üçüncü kod ilk iki kodun birleşimi gibi sürekli powershell kullanmak yerine .bat uzantılı bir dosya kullanmanıza imkan sağlıyor. Dördüncü kod ise Windows defender üzerinden msedgewebview2'ni internet erişimini engelliyor.

Tüm kodların dezavantajı mevcut, kodlar ve engellemeler vs olmadan tek bir yöntem buldum, Autocad'in içine girdiğinizde kendi kod satırına STARTMODE yazın ve ekrana 0 değerini girip yeniden başlatın, Autocad'in standart başlangıç ekranına erişemeyeceksiniz ancak bu sayede pop-up ekranıyla da karşılaşmıyorsunuz, bunu yapabilmek için öncelikle aşağıdaki ilk kodu çalıştırmanız gerekecek, çünkü pop-up yüzünden herhangi bir işlem yapılamıyor, ancak bu yöntem diğerlerine göre oldukça iyi ve dezavantajı bana göre hiç yok, proje oluşturma, dosyaları çalıştırma vb. şeylerde ve Autocad'in standart özelliklerinin herhangi birinde bir sorunla karşılaşmadan çalışamalarınızı yürütebilirsiniz.



Kodlar ChatGPT tarafından oluşturulmuştur.

----------------------------------------------------------------------------------------------------------------------- 

İlk powershell kodu, bu kodu Autocad açıkken kullanarak pop-up ekranını engellersiniz

Get-Process msedgewebview2 -ErrorAction SilentlyContinue | Stop-Process -Force

----------------------------------------------------------------------------------------------------------------------- 

ikinci powershell kodu, bu kodu çalıştırdığında Autocad açılır ve pop-up ekranı engellenir

    $acad = (Get-ItemProperty "HKLM:\SOFTWARE\Autodesk\AutoCAD\R24.3\ACAD-*\").AcadLocation
    Start-Process "$acad\acad.exe"
    Start-Sleep -Seconds 2
    Get-Process msedgewebview2 -ErrorAction SilentlyContinue | Stop-Process -Force
    $acad = (Get-ItemProperty "HKLM:\SOFTWARE\Autodesk\AutoCAD\R24.3\ACAD-*\").AcadLocation
    Start-Process "$acad\acad.exe"
    Start-Sleep -Seconds 2
    Get-Process msedgewebview2 -ErrorAction SilentlyContinue | Stop-Process -Force
-----------------------------------------------------------------------------------------------------------------------

powershell yerine .bat olarak daha kolay bir versiyonu isterseniz bunu kullanabilirsiniz, bu kodu .bat haline getirin ve tıklayınca msedgewebview2'yi engelleyip Autocad'i başlatır, dezavantajı her başlatırken kullanmanız gerek.

İşlem
Buradaki kodu bir metin belgesine kopyalayıp belgeyi farklı kaydet diyerek batch dosyası olarak kaydedeceğiz yani .bat olarak uzantıya sahip olmalı. Sonra çalıştırın ve bu kadar.

    @echo off
    powershell -NoProfile -ExecutionPolicy Bypass -Command "$acad = (Get-ItemProperty 'HKLM:\SOFTWARE\Autodesk\AutoCAD\R24.3\ACAD-*').AcadLocation; Start-Process \"$acad\acad.exe\"; Start-Sleep -Seconds 2; Get-Process msedgewebview2 -ErrorAction SilentlyContinue | Stop-Process -Force"
-----------------------------------------------------------------------------------------------------------------------

Windows defender üzerinden msedgewebview2'nin internet ulaşımını kesmek isterseniz .bat olarak bu kodu kaydedip çalıştırabilirsiniz, tek seferlik yapmanız yeterlidir, güncellemeler yüzünden bir iki ay içinde tekrar yapmanız gerekebilir ama daha büyük bir dezavantajı var o da Outlook, WhatsApp gibi uygulamaların internet erişimi de buradan sağlanıyor ve o uygulamaları bu yüzden kullanamıyorsunuz. Bu kod yerine manuel olarak da Windows defender üzerinden internet erişimini engelleyebilirsiniz.

İşlem
Buradaki kodu bir metin belgesine kopyalayıp belgeyi farklı kaydet diyerek batch dosyası olarak kaydedeceğiz yani .bat olarak uzantıya sahip olmalı. Sonra çalıştırın ve bu kadar.

    @echo off
    net session >nul 2>&1
    if %errorLevel% neq 0 (
        powershell "Start-Process '%~f0' -Verb RunAs"
        exit /b
    )
    
    set base="C:\Program Files (x86)\Microsoft\EdgeWebView\Application"
    
    echo Kurallar ekleniyor...
    
    for /d %%D in (%base%\*) do (
        if exist "%%D\msedgewebview2.exe" (
            set ver=%%~nxD
    
            echo Inbound engelleniyor: %%D\msedgewebview2.exe
            netsh advfirewall firewall add rule name="Block_WebView2_In_!ver!" dir=in action=block program="%%D\msedgewebview2.exe" enable=yes profile=any
    
            echo Outbound engelleniyor: %%D\msedgewebview2.exe
        netsh advfirewall firewall add rule name="Block_WebView2_Out_!ver!" dir=out action=block program="%%D\msedgewebview2.exe" enable=yes profile=any
        )
    )
    
    echo TAMAMLANDI.
    exit
