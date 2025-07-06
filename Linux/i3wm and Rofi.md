```bash
sudo apt update
sudo apt install i3 rofi i3status i3lock suckless-tools feh fonts-font-awesome

```


nel login cambiare environment 


|Azione|Tasto (Mod = Win o Alt)|
|---|---|
|Avvia terminale|`Mod + Enter`|
|Chiudi finestra|`Mod + Shift + Q`|
|Rilancia rofi (app menu)|`Mod + D`|
|Muovi tra finestre|`Mod + frecce`|
|Dividi in verticale|`Mod + V`|
|Dividi in orizzontale|`Mod + H`|
|Modalità floating|`Mod + Shift + Space`|
|Ricarica config|`Mod + Shift + R`|

## Aggiungere Rofi al config di i3

Apri il file config:


```nano
vi ~/.config/i3/config
```

Cerca questa riga (o simile):


```bash
bindsym $mod+d exec dmenu_run
```


Sostituiscila con:

```bash
bindsym $mod+d exec rofi -show drun
```

Poi salva 

Ricarica con:

```bash
Mod + Shift + R
```

Ora `Mod + D` ti apre **rofi** per avviare app come un menu.

## Aggiungi uno sfondo

Scarica lo sfondo da https://wallhaven.cc/

e aggiungi nel file `~/.config/i3/config` questa riga 

```bash
exec --no-startup-id feh --bg-scale /percorso/della/tua/immagine.jpg
```

