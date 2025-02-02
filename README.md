if(!$("#planer_klinow").length){
    var konfiguracja = konfiguracjaSwiata();

    var dane = {
        predkosc_gry: Number($(konfiguracja).find("config speed").text()),
        predkosc_jednostek: Number($(konfiguracja).find("config unit_speed").text()),
        linkDoWojska: "/game.php?&village=" + game_data.village.id + "&type=own_home&mode=units&group=0&page=-1&screen=overview_villages",
        linkDorozkazu: "/game.php?&village=",
        predkosc_szlachcic: 35, // Prędkość szlachcica
    };

    var mojeWioski = [];
    var nazwyWiosek = [];
    var id = [];
    var czasWyjscia = [];

    var predkosc_swiata = Number((dane.predkosc_gry * dane.predkosc_jednostek).toFixed(5));
    dane.predkosc_szlachcic /= predkosc_swiata;

    rysujPlaner();
    pobierzDane();
} else {
    $("#planer_klinow").remove();
}

function rysujPlaner(){
    var elem = "<div id='planer_klinow' class='vis vis_item'>";
    elem += "<h3>Planowanie ataku szlachcicem</h3>";
    elem += "<table id='lista_wiosek'><thead><tr><th>Wioska</th><th>Cel (X|Y)</th><th>Data</th><th>Godzina</th><th>Oblicz</th></tr></thead><tbody>";

    for(let i=0; i<mojeWioski.length; i++){
        elem += "<tr>";
        elem += "<td>" + nazwyWiosek[i] + "</td>";
        elem += "<td><input type='text' id='cel_"+i+"' size='7'></td>";
        elem += "<td><input type='text' id='data_"+i+"' size='10'></td>";
        elem += "<td><input type='text' id='godzina_"+i+"' size='8'></td>";
        elem += "<td><button onclick='obliczCzas("+i+")'>Oblicz</button></td>";
        elem += "</tr>";
    }

    elem += "</tbody></table>";
    elem += "</div>";
    $("#contentContainer").prepend(elem);
}

function obliczCzas(index){
    var cel = $("#cel_"+index).val().match(/\d+/g);
    var dataWejscia = $("#data_"+index).val().match(/\d+/g);
    var godzinaWejscia = $("#godzina_"+index).val().match(/\d+/g);
    
    if(!cel || !dataWejscia || !godzinaWejscia) {
        alert("Podaj poprawne współrzędne, datę i godzinę!");
        return;
    }

    var obecnyCzas = new Date();
    var czasWejscia = new Date(dataWejscia[2], dataWejscia[1] - 1, dataWejscia[0], godzinaWejscia[0], godzinaWejscia[1], godzinaWejscia[2]);
    var roznicaSekund = (czasWejscia - obecnyCzas) / 1000;

    var a = Math.abs(Number(cel[0]) - mojeWioski[index][0]);
    var b = Math.abs(Number(cel[1]) - mojeWioski[index][1]);
    var czasPrzejscia = Math.sqrt((a * a) + (b * b)) * dane.predkosc_szlachcic * 60;

    var czasWyjscia = new Date(czasWejscia - czasPrzejscia * 1000);
    alert("Czas wyjścia z wioski: " + czasWyjscia.toLocaleString());
}

function pobierzDane(){
    var r = new XMLHttpRequest();
    r.open('GET', dane.linkDoWojska, true);
    r.onreadystatechange = function () {
        if (r.readyState == 4 && r.status == 200) {
            var requestedBody = document.createElement("body");
            requestedBody.innerHTML = r.responseText;
            var tabela = $(requestedBody).find('#units_table').get()[0];
            if (!tabela) return;
            
            for (let i = 1; i < tabela.rows.length; i++) {
                id.push(tabela.rows[i].cells[0].getElementsByTagName('span')[0].getAttribute("data-id"));
                mojeWioski.push(tabela.rows[i].cells[0].getElementsByTagName('span')[2].textContent.match(/\d+/g));
                nazwyWiosek.push(tabela.rows[i].cells[0].getElementsByTagName('span')[2].textContent);
            }
            rysujPlaner();
        }
    };
    r.send(null);
}

function konfiguracjaSwiata(){
    var dt;
    $.ajax({
        'async': false,
        'url': '/interface.php?func=get_config',
        'dataType': 'xml',
        'success': function(data){ dt = data; }
    });
    return dt;
}
