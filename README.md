<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <title>US–Iran War: Oil Facility Attacks Map (Carto Positron)</title>
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css" />
  <style>
    body { margin: 0; padding: 0; font-family: Arial, sans-serif; }
    #map { position: absolute; top: 0; left: 0; width: 100%; height: 100%; }
    .legend {
      position: absolute;
      bottom: 20px;
      right: 20px;
      background: white;
      padding: 10px;
      border-radius: 5px;
      box-shadow: 0 0 10px rgba(0,0,0,0.1);
      font-size: 12px;
      z-index: 9999;
    }
    .legend strong { display: block; margin-bottom: 5px; }
    .legend span { display: block; margin: 4px 0; }
  </style>
</head>
<body>
  <div id="map"></div>
  <div class="legend">
    <strong>Legend</strong>
    <span>🟢 US/Coalition strike</span>
    <span>🔴 Iranian strike</span>
    <span>🟡 Naval / tanker incident</span>
  </div>

  <script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
  <script>
    const map = L.map("map").setView([26.5, 51], 5);

    L.tileLayer("https://{s}.basemaps.cartocdn.com/light_all/{z}/{x}/{y}{r}.png", {
      attribution: '&copy; OpenStreetMap contributors &copy; CARTO',
      subdomains: "abcd",
      maxZoom: 20
    }).addTo(map);

    let attacks = [
      { lat: 35.7074, lng: 51.4012, name: "Tehran oil depots", type: "coalition", date: "2026-03-08", desc: "Coalition strikes on 5 oil storage sites." },
      { lat: 25.5, lng: 56.5, name: "Strait of Hormuz tanker incident", type: "naval", date: "2026-03-12", desc: "Iranian fast boats harassed tanker." },
      { lat: 29.0, lng: 50.3, name: "Kharg Island oil terminal", type: "coalition", date: "2026-03-13", desc: "US precision strike on naval/missile infrastructure." },
      { lat: 24.0, lng: 54.5, name: "UAE offshore tanker incident", type: "naval", date: "2026-03-15", desc: "Explosive drone boat damaged UAE tanker." },
      { lat: 27.5, lng: 51.5, name: "South Pars / Asaluyeh complex", type: "coalition", date: "2026-03-18", desc: "Strike hit gas processing and pipelines." },
      { lat: 26.5, lng: 50.5, name: "Ras Laffan LNG complex, Qatar", type: "iranian", date: "2026-03-18", desc: "Iranian missile/drone strikes caused extensive damage." },
      { lat: 24.25, lng: 39.0, name: "SAMREF refinery, Yanbu, Saudi Arabia", type: "iranian", date: "2026-03-19", desc: "Iranian drone attack caused limited damage." },
      { lat: 26.2, lng: 50.6, name: "Bahrain Sitra refinery", type: "iranian", date: "2026-03-20", desc: "Iranian drones targeted Bahrain’s main refinery." },
      { lat: 23.7, lng: 54.3, name: "Habshan Gas Complex, UAE", type: "iranian", date: "2026-03-20", desc: "Missile debris forced shutdown of UAE’s massive gas facility." },
      { lat: 29.3, lng: 47.9, name: "Mina al-Ahmadi terminal, Kuwait", type: "iranian", date: "2026-03-21", desc: "Missile strike disrupted crude exports." },
      { lat: 23.6, lng: 58.6, name: "Qalhat LNG plant, Oman", type: "iranian", date: "2026-03-22", desc: "Drone swarm damaged storage tanks." },
      { lat: 30.5, lng: 47.8, name: "Rumaila oil field, Iraq", type: "coalition", date: "2026-03-23", desc: "Coalition strike on militia positions near oil infrastructure." },
      { lat: 25.9, lng: 49.7, name: "Abqaiq oil processing facility, Saudi Arabia", type: "iranian", date: "2026-03-24", desc: "Missile barrage caused partial shutdown." },
      { lat: 25.1, lng: 56.3, name: "Fujairah tanker anchorage, UAE", type: "naval", date: "2026-03-25", desc: "Explosive-laden boat damaged anchored tanker." },
      { lat: 27.2, lng: 56.3, name: "Bandar Abbas naval base, Iran", type: "coalition", date: "2026-03-26", desc: "Strike on missile launchers near oil export facilities." },
      { lat: 32.65, lng: 51.67, name: "Isfahan Heavy Water Plant", type: "coalition", date: "2026-03-27", desc: "Israeli strike on heavy water facility." },
      { lat: 32.7, lng: 51.6, name: "Yellowcake Production Plant", type: "coalition", date: "2026-03-27", desc: "US/Israel strike on yellowcake facility." },
      { lat: 32.65, lng: 51.65, name: "Esfahan Steel Company", type: "coalition", date: "2026-03-27", desc: "Airstrike hit one of Iran’s largest steelmakers." },
      { lat: 32.55, lng: 51.5, name: "Mobarakeh Steel Complex", type: "coalition", date: "2026-03-27", desc: "Coalition strike targeted major steel facility." },
      { lat: 23.5, lng: 45.0, name: "Saudi Base (unspecified)", type: "iranian", date: "2026-03-27", desc: "Iran launched missiles at a Saudi base." }
    ];

    // Sort chronologically
    attacks.sort((a, b) => new Date(a.date) - new Date(b.date));

    const icons = {
      coalition: L.icon({
        iconUrl: "https://cdnjs.cloudflare.com/ajax/libs/leaflet/1.9.4/images/marker-icon.png",
        shadowUrl: "https://cdnjs.cloudflare.com/ajax/libs/leaflet/1.9.4/images/marker-shadow.png",
        iconSize: [25, 41],
        iconAnchor: [12, 41],
        popupAnchor: [1, -34]
      }),
      iranian: L.divIcon({ className: "emoji-marker", html: "🔴", iconSize: [25, 25], iconAnchor: [12, 12] }),
      naval: L.divIcon({ className: "emoji-marker", html: "🟡", iconSize: [25, 25], iconAnchor: [12, 12] }),
      coalition: L.divIcon({ className: "emoji-marker", html: "🟢", iconSize: [25, 25], iconAnchor: [12, 12] })
    };

    // Add markers with popup + chronological number tooltip (no date)
    attacks.forEach((a, index) => {
      L.marker([a.lat, a.lng], { icon: icons[a.type] })
        .addTo(map)
        .bindPopup(`<b>${a.name}</b><br>Date: ${a.date}<br>${a.desc}`)
        .bindTooltip(`${index + 1}`, { permanent: true, direction: "top" });
    });
  </script>
</body>
</html>
