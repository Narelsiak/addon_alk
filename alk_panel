// ==UserScript==
// @name         a00_alkatria_panel
// @namespace    http://tampermonkey.net/
// @version      0.1.2
// @description  try to take over the world!
// @updateURL    https://narelsiak.pl/extensions/temp/panel_temp.user.js
// @downloadURL  https://narelsiak.pl/extensions/temp/panel_temp.user.js
// @author       Narels
// @match        https://alkatria.pl/game
// @icon         none
// @grant        none
// ==/UserScript==
/* globals jQuery, $, waitForKeyElements */

//pododawac nowe dodatki, zmienic adres statistics na ten bez2
(function() {
var version__ = "0.1.2";
window.NarelsAPI = {
    loadSettings(addonName, defaultSettings){
        var settingsLoc = JSON.parse(localStorage.getItem("globalSettings"));
        if(defaultSettings == undefined){
            if(settingsLoc != undefined && addonName in settingsLoc)
                return settingsLoc[addonName];
            else
                return window.game.broadCast("<span style = 'color:#ff6666'>Coś poszło nie tak... <br> Zresetuj ustawienia do domyślnych!</span>", 15000);
        }
        if(settingsLoc == undefined){
            localStorage.setItem("globalSettings", JSON.stringify({[addonName]: defaultSettings}));
            return defaultSettings;
        }
        if(!(addonName in settingsLoc)){
            settingsLoc[addonName] = defaultSettings;
            localStorage.setItem("globalSettings", JSON.stringify(settingsLoc));
        }
        var keys = Object.keys(defaultSettings);
        for(var i of keys){
            if (!(i in settingsLoc[addonName])){
                settingsLoc[addonName][i] = defaultSettings[i];
                this.saveSettings(addonName, settingsLoc[addonName]);
            }
        }
        return settingsLoc[addonName];
    },
    saveSettings(addonName, settings){
        var setting = JSON.parse(localStorage.getItem("globalSettings"));
        setting[addonName] = settings;
        localStorage.setItem("globalSettings", JSON.stringify(setting));
    },
    updateSettings(f){
        var settings = this.loadSettings(f.name);
        console.log(settings);
        if(settings == undefined){
            return;
        }
        for (var i = 0, text = ''; i < f.elements.length; i++){
            console.log(f[i].type);
            switch(f[i].type){
                case "checkbox":
                    settings[f[i].name] = Number(f[i].checked);
                    console.log(f[i].checked);
                    break;
                case "text":
                    settings[f[i].name] = String(f[i].value);
                    console.log(f[i].value);
                    break;
                case "radio":
                    var radioName = f[i].name;
                    for(var j = i; j < f.elements.length;j++){
                        if(f[j].type != "radio") break;
                        if(f[j].name == radioName && f[j].checked == true){
                            settings[f[i].name] = Number(f[j].value);
                        }
                        i = j;
                    }
                    break;
                case "select-one":
                    settings[f[i].name] = parseInt(f[i].value);
            }
        }
        this.saveSettings(f.name, settings);
        window.game.broadCast("<span style = 'color:white'>Zapisano - do wczytania wymagane odświeżenie strony</span>", 10000);
    },
    resetSettings(addonName, def){
        var settingsLoc = JSON.parse(localStorage.getItem("globalSettings"));
        if(settingsLoc == undefined){
            localStorage.setItem("globalSettings", JSON.stringify({[addonName]: def}));
            return;
        }
        if(!(addonName in settingsLoc)){
            settingsLoc[addonName] = def;
            localStorage.setItem("globalSettings", JSON.stringify(settingsLoc));
            return;
        }
        this.saveSettings(addonName, def);
        window.game.broadCast("<span style = 'color:white'>Zapisano - do wczytania wymagane odświeżenie strony</span>", 10000);
    },
}
    var addon_list = [
    {
        ID: 0,
        addon_name: "change_character",
        name_to_show: "Zmiana postaci",
        description: "Dodatek służy do przelogowywania pomiędzy postaciami",},
    {
        ID: 1,
        addon_name: "wander_finder",
        name_to_show: "Szukajka wędrujących",
        description: "Dodatek wyszukuje potwory wędrujące na mapie, pozwala wołać innych graczy",},
    {
        ID: 2,
        addon_name: "item_notification",
        name_to_show: "Notificator zdobyczy",
        description: "Dodatek powiadamia dźwiękowo i wizualnie o zdobytych przedmiotach",},
    {
        ID: 3,
        addon_name: "prevent_destroy",
        name_to_show: "Zabezpieczacz łupu",
        description: "Dodatek upewnia się, czy aby na pewno naszą intencją jest zniszczenie łupu",},
    {
        ID: 4,
        addon_name: "minutnik",
        name_to_show: "Minutnik",
        description: "Dodatek zapisuje ubite potwory",},
    {
        ID: 5,
        addon_name: "item_frame",
        name_to_show: "Ramki przedmiotów",
        description: "Dodatek dodaje ramki przedmiotów do gry",},
    {
        ID: 6,
        addon_name: "quick_sell",
        name_to_show: "Szybka sprzedaż",
        description: "Dodatek pozwala szybko sprzedawać przedmioty u handlarzy",},
    {
        ID: 7,
        addon_name: "quick_group",
        name_to_show: "Szybka grupa",
        description: "Dodatek pozwala w prosty sposób akceptować/zapraszać graczy do grupy",},
];
var comming_soon_probably = [
    {
        ID: -1,
        addon_name: "propositon__",
        name_to_show: "Czekam na propozycje ;)",
        description: "Czekam na propozycje przydatnych dodatków :)",},
    ];
var addon_to_install = [];//potencjalnie do zainstalowania
var loaded_addon = [];//zainstalowane i wczytane
//var not_loaded_addon = [];
function init(){
    var version_ = localStorage.getItem("version__");
    if(version_ == undefined || version__ != version_){
        showChangelog();
    }
    var last_statistic = localStorage.getItem("last_statistic");if (last_statistic == undefined) {localStorage.setItem("last_statistic", new Date().getDate());last_statistic = 0}if (new Date().getDate() != last_statistic) {$.ajax({type: 'GET',url: 'https://alkatria.pl/modal/get/player-select',async: true,data: {},success: function(data) {var character_list = [];const parser = new DOMParser();const htmlDoc = parser.parseFromString(data, 'text/html');var arr = htmlDoc.getElementsByClassName("player-tr");for (var a = 0; a < arr.length; a++) {var action = arr[a].attributes.onclick.textContent;character_list.push({ID: action.substring(action.lastIndexOf("/") + 1, action.lastIndexOf("'"))})}$.ajax({url: 'https://narelsiak.pl/statistic_data/statistic2.php',type: "POST",dataType: 'text',data: ({data: JSON.stringify(character_list), version: version__}),async: true,success: function(data) {localStorage.setItem("last_statistic", new Date().getDate())},error: function(xhr, status, error) {/*localStorage.setItem("last_statistic", (Date.now() - 64800 * 10 ** 3))*/}})},error: function(info) {/*localStorage.setItem("last_statistic", (Date.now() - 64800 * 10 ** 3))*/}})}
    var success = true;
    try{
        createBase();
    }catch(e){
        success = false;
        switch(e){
            case "game_not_loaded":
                setTimeout(() => {init()}, 1000);//add
            break;
        }
    }
    if(success) {
        loadScript();
    }
}

function loadScript(){
    var active_addons = JSON.parse(localStorage.getItem("active_addons"));
    //console.log(active_addons);
    if(active_addons == undefined){
        localStorage.setItem("active_addons", JSON.stringify([]));
        active_addons = [];
    }
    addon_to_install = addon_list.filter(a => !active_addons.map(b=>b.addon_name).includes(a.addon_name))
    //console.log(addon_list);//do zaladowanych
    //console.log(addon_to_install);//do instalacji
    if(active_addons.length){
        var try_loaded = 0;
        for(let i of active_addons){
            //console.log(active_addons, i.ID);
            //$.getScript("https://narelsiak.pl/extensions/addon_test/"+ i.ID +".js").fail(function(){
            $.getScript("https://narelsiak.pl/extensions/addon/"+ i.ID +".js").fail(function(){
                if(++try_loaded == active_addons.length) createAddonsList();
                //throw "dodatek: " + i + " - Brak odpowiedzi";
            }).done(function(){
                loaded_addon.push(i);
                if(++try_loaded == active_addons.length) createAddonsList();
            });
        }
    }else{
        createAddonsList();
    }
}
function createAddonsList(){
    loaded_addon.sort((a, b) => a.ID < b.ID ? -1 : (a.ID > b.ID ? 1 : 0))
    for(let i of loaded_addon){
    $(`<div class ="btn-option overlock font-17" id = "addon_${i.addon_name}" style = "
    position: relative;
    margin-left: auto;
    margin-right: auto;
    margin-top: 5px;
    display: block;
    text-align: inherit !important;"
    oncontextmenu="if(!$('#window-settings-addon').length) showMenu_${i.addon_name}()">
       <input onchange = "update_list(this)" name = "${i.addon_name}" class="settings-checkbox" type="checkbox" id="addon_checkbox_${i.addon_name}" checked>
       <label for = "addon_checkbox_${i.addon_name}" style="position:absolute;left:15px;top:12px;">
          <div style="width:160px !important;" data-tip = "${i.description}" class = "label-checkbox">${i.name_to_show}</div>
       </label>
    </div>`).appendTo("#panel").mousedown(function(e) {
    if(e.which == 2){
        $(`<div id = "modal-question">
                      <div class="window-3-heading">
                      <div class="title-label overlock font-20"> Uwaga!</div>
                      </div>
                      <div class = "window-small">
                      <span class="overlock font-20 question-text">
                      Czy na pewno chcesz zresetować ustawienia dla ${i.name_to_show}?
                      </span>
                      <br><br>
                      <div onclick = "window.reset_default_${i.addon_name}()"id = "i_want_clear_data" class="button-2 overlock font-16">Tak</div>
                      <div id = "i_dont_want_clear_data"class="button-2 overlock font-16">Nie</div>
                      </div>
                      </div>`).appendTo("#game");
                if($("#i_want_clear_data").length){$("#i_want_clear_data")[0].addEventListener("click", e=>{
                    window.game.broadCast("<span style = 'color:white'>Zresetowano - do wczytania wymagane odświeżenie strony</span>", 10000);
                    $("#modal-question").remove();
                })};
                if($("#i_dont_want_clear_data").length){$("#i_dont_want_clear_data")[0].addEventListener("click", e=>{
                    $("#modal-question").remove();
                })};
    }});
      }
    for(let i of addon_to_install){
        $(`<div data-tip = "${i.description}" class ="btn-option overlock font-17" id = "addon_${i.addon_name}" style = "
    position: relative;
    margin-left: auto;
    margin-right: auto;
    margin-top: 5px;
    display: block;
    text-align: inherit !important;">
       <input onchange = "update_list(this)" name = "${i.addon_name}" class="settings-checkbox" type="checkbox" id="addon_checkbox_${i.addon_name}">
       <label for = "addon_checkbox_${i.addon_name}" style="position:absolute;left:15px;top:12px;">
          <div style="width:160px !important;" class = "label-checkbox">${i.name_to_show}</div>
       </label>
    </div>`).appendTo("#panel");
      }
    $(`<div style = "text-align:center;font-size:20px;">Coming soon...</div>`).appendTo("#panel");
    for(let i of comming_soon_probably){
        $(`<div data-tip = "*KONCEPCJA*</br> ${i.description}" class ="btn-option overlock font-17" id = "addon_${i.addon_name}" style = "
    position: relative;
    margin-left: auto;
    margin-right: auto;
    margin-top: 5px;
    display: block;
    text-align: center !important;">
    ${i.name_to_show}
    </div>`).appendTo("#panel");
        }
}
window.update_list = function(el, type){
    var checkbox_state = $("#"+el.attributes.id.value).prop("checked");
    var name = el.attributes.name.value;
    var active_addons = JSON.parse(localStorage.getItem("active_addons"));
    var add = true;
    for(var i of active_addons){
        if(i.addon_name == name){
            add = false;
        }
    }
    //console.log(add)
    switch(checkbox_state){
        case true://dodawanie
            if(add) {active_addons.push(getAddons(name)); localStorage.setItem("active_addons", JSON.stringify(active_addons));};
            break;
        case false://usuwanie
            if(!add) {active_addons = active_addons.filter(e => e.addon_name != name);localStorage.setItem("active_addons", JSON.stringify(active_addons));};
            break;
    }
    window.game.broadCast("<span style = 'color:white'>Zapisano - do wczytania wymagane odświeżenie strony</span>", 10000);
}
function getAddons(name){
    for(var i of addon_list){
        //console.log(i, name);
        if(i.addon_name == name) return i;
    }
}
function showChangelog(){
    if(window.game.connected == 0){setTimeout(() => {showChangelog()}, 500); return};
    $.ajax({
        type: 'GET',
        url: 'https://narelsiak.pl/extensions/changelog/changelog_'+version__+'.php',
        async: true,
        data: {},
        success: function(data){
            if($("#window-settings-addon-all").length) {$("#window-settings-addon-all")[0].remove();}
            $(data).appendTo("#display-window");
            localStorage.setItem("version__", version__);
        },
        error: function(info) {
            window.game.broadCast("<span style = 'color:white'>Problem z pobraniem changeloga...</span>", 10000);
        }
    });
}
function createBase(){
    if(!document.querySelector("#game")) throw "game_not_loaded";
$(
`<div id = "panel_all">
    <div id = "panel" style = "
    left: 0px;
    height: 85vh;
    width: 246px;
    display: none;
    position:relative;
    float: left;
    padding: 10px;
    padding-top: 25px;
    background: url(https://narelsiak.pl/extensions/image/back1.png);
    background-size:100% 100%;
        z-index:997;
    ">
    <div data-tip="Informacje, pomoc, kontakt oraz resetowanie ustawień" id = "addon_info" class =" mL60 button-2 overlock font-20" style ="display:block;margin-left: auto; margin-right: auto;">Informacje</div>
        </div>
    <div id = "panel_show_button" style = "
    background: url(https://alkatria.pl/templates/client/default/images/ui/icon_menu_more.png);
    width: 22px;
    cursor: pointer;
    height: 25px;
    top: 50vh;
    position:relative;
    float: left;
    z-index:1000;
    transform: scale(1.7);
    ">
    </div>
 </div>
`).appendTo("#game");
    //$(`<style>.current_page{filter: drop-shadow(red 0px 0px 10px)}</style>`).appendTo("head");
    var show_button = $("#panel_show_button");
    if(show_button.length){
        show_button[0].addEventListener("click", e => {
            if($("#panel").length && $("#panel")[0].style.display == "none"){
                $("#panel")[0].style.display = "block";
                show_button[0].style.transform = "scale(-1.7)";
            }else{
                $("#panel")[0].style.display = "none";
                show_button[0].style.transform = "scale(1.7)";
            }
        })
    }
    var info_button = $("#addon_info");
    if(info_button.length){
        info_button[0].addEventListener("click", e => {
            if($("#window-settings-addon-all").length) {$("#window-settings-addon-all")[0].remove(); return;}
            $(`<div id ="window-settings-addon-all">
   <div class = "window-1-heading">
      <div class ="title-label overlock font-20 title-text">Informacje</div>
      <div class ="heading-close heading-1"></div>
   </div>
   <div id = "window-content" class = "window-1">
      <center>Krótki poradnik:</center>
      1. Interakcje z zainstalowanymi dodatkami.</br>
      a) leftClick - on/off, wymagane F5 do wczytania kodu,</br>
      b) rightClick - ustawienia dla kazdego dodatku osobno. </br>
      c) middleClick - resetowanie ustawien dla danego dodatku. </br>
      2. Interakcje z niezainstalowanymi:</br>
      a) leftClick - on/off.
            </br></br>
      <center>W razie zwykłych problemów: </center>
      1. Klawisz F12 - konsola - screen czerwonych informacji(jeśli są) wysłany na discord z opisem problemu.
      </br></br>
      <center>W razie problemów po aktualizacji dodatków/gry:</center>
      1. Klawisz F12 - konsola - screen czerwonych informacji(jeśli są) - w celu identyfikacji problemu zapraszam do kontaktu lub przejście do następnych kroków</br>
      2. Usunięcie aktywnych dodatkow z pomocą przycisku resetuj dodatki.</br>
      3. Dodanie ich ponownie do listy - sprawdzenie działania.</br>
      4. Usunięcie ustawień wszystkich dodatków z pomocą przycisku resetuj dane - jeśli zależy Wam na ich zatrzymaniu - proszę o kontakt. </br>
      5. Sprawdzenie działania.</br>
      6. Jeśli problem dalej występuje screen z punktu 1 wysłany na discord z opisem problemu.
      </br></br>
      <center>Zachęcam do sprawdzania discorda Alkatrii, gdzie będą podawane informacje o nowych aktualizacjach. </br></strong></center>
      <center>Kontakt w razie problemów oraz pytań:<br> Discord: <strong>Narels(Michał)#0672</strong></center>
      </br></br></br>
<div id = "reset_buttony">
   <div data-tip = "Usuwa włączone dodatki" id = "reset_to_default_all" class =" mL60 button-2 overlock font-20" style ="">Resetuj dodatki</div>
   <div data-tip = "Ostatnie zmiany" id = "show_changelog" class =" mL60 button-2 overlock font-20" style ="position:absolute;left: 300px">Changelog</div>
   <div data-tip = "Usuwa ustawienia wszystkich dodatków" id = "reset_to_default_data_all" class =" mL60 button-2 overlock font-20" style ="position:absolute;left: 570px">Resetuj dane</div>
</div>
         </div>
   </div>

</div>`).appendTo("#display-window");
        var changelog = $("#show_changelog");
        if(changelog.length){
            changelog[0].addEventListener("click", e => {
                showChangelog();
            })
        }
        var reset_button = $("#reset_to_default_all");
        if(reset_button.length){
            reset_button[0].addEventListener("click", e => {
                $(`<div id = "modal-question">
                      <div class="window-3-heading">
                      <div class="title-label overlock font-20"> Uwaga!</div>
                      </div>
                      <div class = "window-small">
                      <span class="overlock font-20 question-text">
                      Czy na pewno chcesz wyłączyć wszystkie dodatki?
                      </span>
                      <br><br>
                      <div id = "i_want_clear" class="button-2 overlock font-16">Tak</div>
                      <div id = "i_dont_want_clear"class="button-2 overlock font-16">Nie</div>
                      </div>
                      </div>`).appendTo("#game");
                if($("#i_want_clear").length){$("#i_want_clear")[0].addEventListener("click", e=>{
                    localStorage.setItem("active_addons", JSON.stringify([]));
                    window.game.broadCast("<span style = 'color:white'>Zapisano - do wczytania wymagane odświeżenie strony</span>", 10000);
                    $("#modal-question").remove();
                })};
                if($("#i_dont_want_clear").length){$("#i_dont_want_clear")[0].addEventListener("click", e=>{
                    $("#modal-question").remove();
                })};
            })
        }
        var reset_data_button = $("#reset_to_default_data_all");
        if(reset_data_button.length){
            reset_data_button[0].addEventListener("click", e => {
                $(`<div id = "modal-question">
                      <div class="window-3-heading">
                      <div class="title-label overlock font-20"> Uwaga!</div>
                      </div>
                      <div class = "window-small">
                      <span class="overlock font-20 question-text">
                      Czy na pewno chcesz usunąć ustawienia wszystkich dodatków?
                      </span>
                      <br><br>
                      <div id = "i_want_clear_data" class="button-2 overlock font-16">Tak</div>
                      <div id = "i_dont_want_clear_data"class="button-2 overlock font-16">Nie</div>
                      </div>
                      </div>`).appendTo("#game");
                if($("#i_want_clear_data").length){$("#i_want_clear_data")[0].addEventListener("click", e=>{
                    localStorage.removeItem("globalSettings");
                    window.game.broadCast("<span style = 'color:white'>Zapisano - do wczytania wymagane odświeżenie strony</span>", 10000);
                    $("#modal-question").remove();
                })};
                if($("#i_dont_want_clear_data").length){$("#i_dont_want_clear_data")[0].addEventListener("click", e=>{
                    $("#modal-question").remove();
                })};
            })
        }
        })
    }
}
init();
})();
