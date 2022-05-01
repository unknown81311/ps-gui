```ps
Add-Type -AssemblyName System.Windows.Forms
Add-Type @"
using System;
using System.Runtime.InteropServices;

public struct RECT {
    public int Left;
    public int Top;
    public int Right;
    public int Bottom;
}

public class WinAPI {
    [DllImport("user32.dll")]
    [return: MarshalAs(UnmanagedType.Bool)]
    public static extern bool GetWindowRect(IntPtr hWnd, out RECT lpRect);
}
"@#colud combine winapi?


$globalIds=New-Object System.Collections.ArrayList

function Make-Button() {#gen a button
    $text = $Args[0]
    $x = $Args[1]
    $y = $Args[2]
    $padding=$Args[3] ? $Args[3] : 0
    $id = (New-Guid).Guid
    $list=@($id,$text,$x,$y,$padding)
    Set-Variable -Name $id -Value @($list) -Scope Global
    $globalIds.Add($id)
    return $id
}

function Render-Button() {#print button text at pos
    $type=$Args[1]
    $Args=(Get-Variable $Args[0]).value
    $text = $Args[1]
    $x = $Args[2]-1
    $y = $Args[3]-1
    $padding=$Args[4]

    $w = $text.Length+2
    #echo $Args
    if (-not $type) {
        [console]::SetCursorPosition($x,$y)
        Write-Host "┌$('─'*($w-2+$padding*2))┐" -NoNewLine
        [console]::SetCursorPosition($x,$y+1)
        Write-Host "│$(' '*($w-2+$padding*2))│" -NoNewLine
        [console]::SetCursorPosition($x+1,$y+1)

        Write-Host "$(' '*$padding)$($text)$(' '*$padding)" -NoNewLine
        [console]::SetCursorPosition($x,$y+2)
        Write-Host "└$('─'*($w-2+$padding*2))┘" -NoNewLine
    } else {
        [console]::SetCursorPosition($x,$y)
        Write-Host "$('▄'*($w+$padding*2))" -NoNewLine
        [console]::SetCursorPosition($x,$y+1)
        Write-Host "$(' '*($padding+1))$($text)$(' '*($padding+1))" -NoNewLine -ForegroundColor $Host.UI.RawUI.BackgroundColor -BackgroundColor $Host.UI.RawUI.ForegroundColor
        [console]::SetCursorPosition($x,$y+2)
        Write-Host "$('▀'*($w+$padding*2))" -NoNewLine
    }
}

$button1 = Make-Button "hello" 20 10 5

function start-gen{
    [System.Windows.Forms.Cursor]::Hide()
    Import-Module "./test.ps1"
    while ($true) {

        $LeftState = [WinApi]::GetAsyncKeyState(1)
    Write-Host "Left mouse state: $LeftState"

        $ClientRect = New-Object RECT

        # get the window rectangle dimensions
        [IntPtr]$Handle = (Get-Process -Id $PID).MainWindowHandle
        [WinAPI]::GetWindowRect($Handle, [ref]$ClientRect) | Out-Null

        # get the console dimensions
        $ConsoleDimensions = $Host.UI.RawUI.WindowSize

        # compute ratio of console dimensions vs. window dimensions
        $RatioWidth = $ConsoleDimensions.Width / ($ClientRect.Right - $ClientRect.Left)
        $RatioHeight = $ConsoleDimensions.Height / ($ClientRect.Bottom - $ClientRect.Top)

        $MousePos = [System.Windows.Forms.Cursor]::Position

        # calculate where mouse is based on position within the console window
        $ConsoleMousePos = [System.Drawing.PointF]::new(($MousePos.X - $ClientRect.Left) * $RatioWidth, ($MousePos.Y - $ClientRect.Top) * $RatioHeight)

        foreach ($ui in $globalIds){
            $button = (Get-Variable "$ui").value
            
            if (([math]::Round($ConsoleMousePos.X+.1) -ge $button[2])-and([math]::Round($ConsoleMousePos.Y+.1) -ge $button[3]+1)-and ([math]::Round($ConsoleMousePos.X+.1) -le $button[2]+$button[1].Length+2+($button[4]*2)-2)-and([math]::Round($ConsoleMousePos.Y+.1) -le $button[3]+2)){
                Render-Button (Get-Variable $button[0]).value[0] 1
            }else{
                Render-Button (Get-Variable $button[0]).value[0] 
            }
        }
        Start-Sleep -Milliseconds 125
    }
}
cls
#Render-Button $button1 1
start-gen
