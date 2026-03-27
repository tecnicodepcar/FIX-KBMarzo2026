# ==============================
# REPARADOR WINDOWS PRO - by ChatGPT (MEJORADO v2)
# ==============================

function Mostrar-Menu {
    Clear-Host
    Write-Host "==============================" -ForegroundColor Cyan
    Write-Host " REPARADOR DE WINDOWS (PRO) "
    Write-Host "==============================" -ForegroundColor Cyan
    Write-Host ""
    Write-Host "1 - Reparar archivos del sistema (SFC + DISM)"
    Write-Host "2 - Reparar permisos del disco (ICACLS)"
    Write-Host "3 - Resetear politicas de seguridad"
    Write-Host "4 - Re-registrar aplicaciones (Apps rotas)"
    Write-Host "5 - Crear nuevo usuario administrador"
    Write-Host "6 - Activar usuario Administrator oculto"
    Write-Host ""
    Write-Host "=== ACTUALIZACIONES PROBLEMATICAS ===" -ForegroundColor Yellow
    Write-Host "7 - Win11: Desinstalar KB5079473 (normal)"
    Write-Host "8 - Win11: Forzar desinstalacion KB5079473"
    Write-Host "9 - Win10: Desinstalar KB5078885 (normal)"
    Write-Host "10 - Win10: Forzar desinstalacion KB5078885"
    Write-Host ""
    Write-Host "=== REPARACION AVANZADA ===" -ForegroundColor Yellow
    Write-Host "11 - Reparacion completa + FIX permisos"
    Write-Host "12 - Reiniciar equipo"
    Write-Host "0 - Salir"
    Write-Host ""
}

function Pausa {
    Write-Host ""
    Read-Host "Presioná ENTER para continuar"
}

# ==============================
# FUNCIONES BASE
# ==============================

function Reparar-Sistema {
    Write-Host "Ejecutando SFC..." -ForegroundColor Yellow
    sfc /scannow

    Write-Host "Ejecutando DISM..." -ForegroundColor Yellow
    DISM /Online /Cleanup-Image /RestoreHealth
}

function Reparar-Permisos {
    Write-Host "Reparando permisos del disco C: ..." -ForegroundColor Yellow
    icacls C:\ /reset /t /c /q
}

function Resetear-Seguridad {
    Write-Host "Reseteando politicas de seguridad..." -ForegroundColor Yellow
    secedit /configure /cfg $env:windir\inf\defltbase.inf /db defltbase.sdb /verbose
}

function Reparar-Apps {
    Write-Host "Re-registrando aplicaciones..." -ForegroundColor Yellow
    Get-AppxPackage -AllUsers | ForEach {
        Add-AppxPackage -DisableDevelopmentMode -Register "$($_.InstallLocation)\AppXManifest.xml"
    }
}

function Crear-Usuario {
    Write-Host "Creando usuario admin 'soporte'..." -ForegroundColor Yellow
    net user soporte 123456 /add
    net localgroup administrators soporte /add
}

function Activar-Admin {
    Write-Host "Activando usuario Administrator..." -ForegroundColor Yellow
    net user administrator /active:yes
}

# ==============================
# KB WINDOWS 11
# ==============================

function Desinstalar-KB-W11 {
    Write-Host "Desinstalando KB5079473 (Windows 11)..." -ForegroundColor Yellow
    wusa /uninstall /kb:5079473 /quiet /norestart
}

function Forzar-KB-W11 {
    Write-Host "Buscando KB5079473..." -ForegroundColor Yellow
    $paquete = dism /online /get-packages | findstr 5079473

    if ($paquete) {
        $nombre = $paquete.Split("|")[-1].Trim()
        dism /online /remove-package /packagename:$nombre /quiet /norestart
        Write-Host "KB5079473 eliminada (forzado)." -ForegroundColor Green
    } else {
        Write-Host "No encontrada." -ForegroundColor Red
    }
}

# ==============================
# KB WINDOWS 10
# ==============================

function Desinstalar-KB-W10 {
    Write-Host "Desinstalando KB5078885 (Windows 10)..." -ForegroundColor Yellow
    wusa /uninstall /kb:5078885 /quiet /norestart
}

function Forzar-KB-W10 {
    Write-Host "Buscando KB5078885..." -ForegroundColor Yellow
    $paquete = dism /online /get-packages | findstr 5078885

    if ($paquete) {
        $nombre = $paquete.Split("|")[-1].Trim()
        dism /online /remove-package /packagename:$nombre /quiet /norestart
        Write-Host "KB5078885 eliminada (forzado)." -ForegroundColor Green
    } else {
        Write-Host "No encontrada." -ForegroundColor Red
    }
}

# ==============================
# FIX AVANZADO
# ==============================

function Fix-Permisos-PostUpdate {
    Write-Host "Aplicando fix de permisos avanzado..." -ForegroundColor Yellow

    takeown /f C:\Windows\System32 /r /d y
    icacls C:\Windows\System32 /grant administrators:F /t

    DISM /Online /Cleanup-Image /StartComponentCleanup

    Write-Host "Fix aplicado." -ForegroundColor Green
}

function Reparacion-Completa {
    Reparar-Sistema
    Reparar-Permisos
    Resetear-Seguridad
    Reparar-Apps
}

function Reparacion-Full-Con-Fix {
    Write-Host "Ejecutando reparación completa + fix..." -ForegroundColor Yellow
    Reparacion-Completa
    Fix-Permisos-PostUpdate
}

function Reiniciar {
    Write-Host "Reiniciando equipo..." -ForegroundColor Yellow
    shutdown /r /t 0
}

# ==============================
# MENU
# ==============================

do {
    Mostrar-Menu
    $opcion = Read-Host "Elegí una opción"

    switch ($opcion) {
        "1" { Reparar-Sistema; Pausa }
        "2" { Reparar-Permisos; Pausa }
        "3" { Resetear-Seguridad; Pausa }
        "4" { Reparar-Apps; Pausa }
        "5" { Crear-Usuario; Pausa }
        "6" { Activar-Admin; Pausa }

        "7" { Desinstalar-KB-W11; Pausa }
        "8" { Forzar-KB-W11; Pausa }
        "9" { Desinstalar-KB-W10; Pausa }
        "10" { Forzar-KB-W10; Pausa }

        "11" { Reparacion-Full-Con-Fix; Pausa }
        "12" { Reiniciar }

        "0" { break }
        default { Write-Host "Opción inválida"; Pausa }
    }

} while ($true)
