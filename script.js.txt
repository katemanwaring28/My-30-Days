"use strict";

/* =========================================================
   MY 30 DAYS
   ========================================================= */

const STORAGE = "my30days_data_v1";

const habitudes = [
  { id: "eau", emoji: "💧", fr: "Boire suffisamment d’eau", en: "Drink enough water", es: "Beber suficiente agua" },
  { id: "matin", emoji: "🌅", fr: "Prendre un bon départ le matin", en: "Have a good start to the morning", es: "Empezar bien la mañana" },
  { id: "repas", emoji: "🍽️", fr: "Prendre un repas dans la journée", en: "Have a meal during the day", es: "Tomar una comida durante el día" },
  { id: "mouvement", emoji: "🧘", fr: "Faire un peu de mouvement", en: "Do some movement", es: "Hacer un poco de movimiento" },
  { id: "pause", emoji: "🌿", fr: "Prendre un moment pour souffler", en: "Take a moment to breathe", es: "Tomarse un momento para respirar" },
  { id: "sommeil", emoji: "🌙", fr: "Préparer une bonne nuit", en: "Prepare for a good night's sleep", es: "Prepararse para dormir bien" }
];

const defis = [
  "Prends quelques minutes pour respirer calmement. 🌿",
  "Écris une chose positive de ta journée. ✨",
  "Écoute une chanson que tu aimes. 🎵",
  "Fais une petite pause loin des écrans. 📵",
  "Prends un moment pour regarder dehors. 🌤️",
  "Fais quelque chose qui te détend. 🧘",
  "Envoie un message gentil à quelqu’un. 💌",
  "Range tranquillement un petit espace. 🧺",
  "Lis quelques pages d’un livre. 📖",
  "Prends un moment pour toi. 💚",
  "Écoute ton corps et accorde-toi une pause si nécessaire. 🌱",
  "Fais une activité créative. 🎨",
  "Passe quelques minutes dehors si tu en as envie. 🌳",
  "Note trois choses que tu apprécies aujourd’hui. ☀️",
  "Fais une activité simplement pour le plaisir. 🎮"
];

const surprises = [
  "Tu n’as pas besoin d’être parfait(e) pour progresser. 🌱",
  "Une petite action compte aussi. ✨",
  "Aujourd’hui peut être un nouveau départ. 🌿",
  "Prends ton temps. Tu avances à ton rythme. 💚",
  "N’oublie pas de faire une pause. 🧘",
  "Tu peux être fier/fière de chaque petit effort. ⭐",
  "Ta journée n’a pas besoin d’être parfaite pour être une bonne journée. ☀️"
];

const traductions = {
  fr: {
    challenge: "MON CHALLENGE",
    day: "Jour",
    habits: "Mes habitudes",
    calendar: "Mon calendrier",
    objective: "Mon objectif",
    wellbeing: "Mon bien-être",
    meals: "Mes repas",
    movement: "Mouvement",
    badges: "Mes badges",
    journal: "Mon journal"
  },
  en: {
    challenge: "MY CHALLENGE",
    day: "Day",
    habits: "My habits",
    calendar: "My calendar",
    objective: "My goal",
    wellbeing: "My wellbeing",
    meals: "My meals",
    movement: "Movement",
    badges: "My badges",
    journal: "My journal"
  },
  es: {
    challenge: "MI RETO",
    day: "Día",
    habits: "Mis hábitos",
    calendar: "Mi calendario",
    objective: "Mi objetivo",
    wellbeing: "Mi bienestar",
    meals: "Mis comidas",
    movement: "Movimiento",
    badges: "Mis insignias",
    journal: "Mi diario"
  }
};

const decorations = {
  printemps: ["🌸", "🌷", "🌼", "🐣", "🐰", "🍓", "🌱"],
  ete: ["☀️", "🌴", "🕶️", "🛟", "🦩", "✈️", "🍉"],
  automne: ["🍂", "🍁", "🎃", "🌰", "🍄", "🌳", "🦊"],
  hiver: ["❄️", "☃️", "⛷️", "🏔️", "🏠", "🧣", "🎿"]
};

let jourActuel = 1;
let musiqueActive = false;
let audioContext = null;
let oscillateur = null;
let gainNode = null;


/* =========================================================
   DONNÉES
   ========================================================= */

function chargerDonnees() {
  try {
    const donnees = localStorage.getItem(STORAGE);

    if (!donnees) {
      return {
        started: false,
        langue: "fr",
        theme: "light",
        saison: "auto",
        vibration: true,
        objectif: "",
        traitement: {
          nom: "",
          heure: ""
        },
        jours: {}
      };
    }

    const parsed = JSON.parse(donnees);

    return {
      started: false,
      langue: "fr",
      theme: "light",
      saison: "auto",
      vibration: true,
      objectif: "",
      traitement: {
        nom: "",
        heure: ""
      },
      jours: {},
      ...parsed
    };

  } catch (erreur) {
    console.error("Erreur localStorage :", erreur);

    return {
      started: false,
      langue: "fr",
      theme: "light",
      saison: "auto",
      vibration: true,
      objectif: "",
      traitement: {
        nom: "",
        heure: ""
      },
      jours: {}
    };
  }
}

let donnees = chargerDonnees();

function sauvegarder() {
  try {
    localStorage.setItem(STORAGE, JSON.stringify(donnees));
  } catch (erreur) {
    console.error("Impossible de sauvegarder :", erreur);
  }
}

function getJour(numero) {
  const cle = String(numero);

  if (!donnees.jours[cle]) {
    donnees.jours[cle] = {
      habitudes: {},
      humeur: "",
      energie: "",
      sommeil: "",
      repas: {
        petitDejeuner: "",
        dejeuner: "",
        diner: "",
        collation: ""
      },
      mouvement: {
        type: "",
        minutes: 0
      },
      journal: ""
    };
  }

  return donnees.jours[cle];
}


/* =========================================================
   DÉMARRAGE
   ========================================================= */

function commencer() {
  donnees.started = true;
  sauvegarder();

  document.getElementById("accueil").classList.add("hidden");
  document.getElementById("challenge").classList.remove("hidden");

  creerCalendrier();
  chargerJour(jourActuel);
  chargerObjectif();
  chargerTraitement();
  creerDecorations();
  mettreAJourStats();

  vibrer();
}

function afficherAccueil() {
  const accueil = document.getElementById("accueil");
  const challenge = document.getElementById("challenge");

  if (donnees.started) {
    accueil.classList.add("hidden");
    challenge.classList.remove("hidden");

    creerCalendrier();
    chargerJour(jourActuel);
  } else {
    accueil.classList.remove("hidden");
    challenge.classList.add("hidden");
  }
}


/* =========================================================
   JOUR / CALENDRIER
   ========================================================= */

function chargerJour(numero) {
  if (numero < 1 || numero > 30) return;

  jourActuel = numero;

  const jour = getJour(numero);

  document.getElementById("numeroJour").textContent = numero;
  document.getElementById("titreJour").textContent =
    `${traductions[donnees.langue].day} ${numero}`;

  afficherDate();
  afficherDefi();
  afficherHabitudes();
  chargerBienEtre();
  chargerRepas();
  chargerMouvement();
  chargerJournal();

  mettreAJourStats();
  mettreAJourCalendrier();
  afficherBadges();
}

function creerCalendrier() {
  const calendrier = document.getElementById("calendrier");

  if (!calendrier) return;

  calendrier.innerHTML = "";

  for (let i = 1; i <= 30; i++) {
    const bouton = document.createElement("button");

    bouton.type = "button";
    bouton.className = "calendar-day";
    bouton.textContent = i;

    bouton.addEventListener("click", () => {
      chargerJour(i);
      window.scrollTo({
        top: 0,
        behavior: "smooth"
      });
    });

    calendrier.appendChild(bouton);
  }

  mettreAJourCalendrier();
}

function mettreAJourCalendrier() {
  const boutons = document.querySelectorAll(".calendar-day");

  boutons.forEach((bouton, index) => {
    const numero = index + 1;
    const jour = getJour(numero);

    bouton.classList.toggle("active", numero === jourActuel);
    bouton.classList.toggle("done", estJourComplete(numero));
  });
}

function estJourComplete(numero) {
  const jour = getJour(numero);

  return habitudes.every(habitude => {
    return jour.habitudes[habitude.id] === true;
  });
}

function afficherDate() {
  const element = document.getElementById("dateJour");

  if (!element) return;

  const maintenant = new Date();

  element.textContent = maintenant.toLocaleDateString(
    donnees.langue === "fr"
      ? "fr-FR"
      : donnees.langue === "es"
        ? "es-ES"
        : "en-US",
    {
      weekday: "long",
      day: "numeric",
      month: "long"
    }
  );
}


/* =========================================================
   DÉFI
   ========================================================= */

function afficherDefi() {
  const element = document.getElementById("defiDuJour");

  if (!element) return;

  const index = (jourActuel - 1) % defis.length;
  element.textContent = defis[index];
}


/* =========================================================
   HABITUDES
   ========================================================= */

function afficherHabitudes() {
  const container = document.getElementById("habitudes");

  if (!container) return;

  const jour = getJour(jourActuel);

  container.innerHTML = "";

  habitudes.forEach(habitude => {
    const label = document.createElement("label");
    label.className = "habit";

    const checkbox = document.createElement("input");

    checkbox.type = "checkbox";
    checkbox.checked = jour.habitudes[habitude.id] === true;

    const langue = donnees.langue;

    const texte =
      langue === "en"
        ? habitude.en
        : langue === "es"
          ? habitude.es
          : habitude.fr;

    const span = document.createElement("span");
    span.textContent = `${habitude.emoji} ${texte}`;

    if (checkbox.checked) {
      label.classList.add("completed");
    }

    checkbox.addEventListener("change", () => {
      jour.habitudes[habitude.id] = checkbox.checked;

      label.classList.toggle("completed", checkbox.checked);

      sauvegarder();
      mettreAJourStats();
      mettreAJourProgression();
      mettreAJourCalendrier();
      afficherBadges();

      if (checkbox.checked) {
        vibrer();
      }
    });

    label.appendChild(checkbox);
    label.appendChild(span);

    container.appendChild(label);
  });

  mettreAJourProgression();
}

function mettreAJourProgression() {
  const jour = getJour(jourActuel);

  const total = habitudes.length;

  const completees = habitudes.filter(
    habitude => jour.habitudes[habitude.id] === true
  ).length;

  const pourcentage = Math.round((completees / total) * 100);

  document.getElementById("progressionTexte").textContent =
    `${pourcentage}%`;

  document.getElementById("progressionBarre").style.width =
    `${pourcentage}%`;
}


/* =========================================================
   STATS
   ========================================================= */

function mettreAJourStats() {
  let joursCompletes = 0;
  let totalHabitudes = 0;

  for (let i = 1; i <= 30; i++) {
    const jour = getJour(i);

    const completees = habitudes.filter(
      h => jour.habitudes[h.id] === true
    ).length;

    totalHabitudes += completees;

    if (completees === habitudes.length) {
      joursCompletes++;
    }
  }

  document.getElementById("joursCompletes").textContent =
    joursCompletes;

  document.getElementById("habitudesCompletes").textContent =
    totalHabitudes;

  const badges = calculerBadges();

  document.getElementById("nombreBadges").textContent =
    badges.filter(b => b.debloque).length;
}


/* =========================================================
   BIEN-ÊTRE
   ========================================================= */

function sauverBienEtre() {
  const jour = getJour(jourActuel);

  jour.humeur = document.getElementById("humeur").value;
  jour.energie = document.getElementById("energie").value;
  jour.sommeil = document.getElementById("sommeil").value;

  sauvegarder();
  vibrer();
}

function chargerBienEtre() {
  const jour = getJour(jourActuel);

  document.getElementById("humeur").value = jour.humeur || "";
  document.getElementById("energie").value = jour.energie || "";
  document.getElementById("sommeil").value = jour.sommeil || "";
}


/* =========================================================
   REPAS
   ========================================================= */

function sauverRepas() {
  const jour = getJour(jourActuel);

  jour.repas = {
    petitDejeuner: document.getElementById("petitDejeuner").value,
    dejeuner: document.getElementById("dejeuner").value,
    diner: document.getElementById("diner").value,
    collation: document.getElementById("collation").value
  };

  sauvegarder();
  vibrer();
}

function chargerRepas() {
  const jour = getJour(jourActuel);

  document.getElementById("petitDejeuner").value =
    jour.repas?.petitDejeuner || "";

  document.getElementById("dejeuner").value =
    jour.repas?.dejeuner || "";

  document.getElementById("diner").value =
    jour.repas?.diner || "";

  document.getElementById("collation").value =
    jour.repas?.collation || "";
}


/* =========================================================
   MOUVEMENT
   ========================================================= */

function choisirMouvement(type) {
  const jour = getJour(jourActuel);

  jour.mouvement.type = type;

  document.getElementById("mouvementChoisi").textContent =
    `Choisi : ${type}`;

  sauvegarder();
}

function sauverMouvement() {
  const jour = getJour(jourActuel);

  const valeur = document.getElementById("minutesMouvement").value;

  let minutes = Number.parseInt(valeur, 10);

  if (!Number.isFinite(minutes) || minutes < 0) {
    minutes = 0;
  }

  if (minutes > 600) {
    minutes = 600;
  }

  jour.mouvement.minutes = minutes;

  sauvegarder();
  vibrer();
}

function chargerMouvement() {
  const jour = getJour(jourActuel);

  document.getElementById("mouvementChoisi").textContent =
    jour.mouvement.type
      ? `Choisi : ${jour.mouvement.type}`
      : "Aucun mouvement sélectionné.";

  document.getElementById("minutesMouvement").value =
    jour.mouvement.minutes || "";
}


/* =========================================================
   BADGES
   ========================================================= */

function calculerBadges() {
  let joursCompletes = 0;

  for (let i = 1; i <= 30; i++) {
    if (estJourComplete(i)) {
      joursCompletes++;
    }
  }

  return [
    {
      emoji: "🌱",
      nom: "Premier jour",
      description: "Compléter ton premier jour",
      debloque: joursCompletes >= 1
    },
    {
      emoji: "⭐",
      nom: "3 jours",
      description: "Compléter 3 jours",
      debloque: joursCompletes >= 3
    },
    {
      emoji: "🔥",
      nom: "7 jours",
      description: "Compléter 7 jours",
      debloque: joursCompletes >= 7
    },
    {
      emoji: "💚",
      nom: "15 jours",
      description: "Compléter 15 jours",
      debloque: joursCompletes >= 15
    },
    {
      emoji: "🏆",
      nom: "30 jours",
      description: "Compléter les 30 jours",
      debloque: joursCompletes >= 30
    }
  ];
}

function afficherBadges() {
  const container = document.getElementById("badges");

  if (!container) return;

  container.innerHTML = "";

  calculerBadges().forEach(badge => {
    const element = document.createElement("div");

    element.className = `badge ${badge.debloque ? "" : "locked"}`;

    element.innerHTML = `
      <span>${badge.emoji}</span>
      <strong>${badge.nom}</strong>
      <p>${badge.description}</p>
    `;

    container.appendChild(element);
  });
}


/* =========================================================
   JOURNAL
   ========================================================= */

function sauverJournal() {
  const jour = getJour(jourActuel);

  jour.journal = document.getElementById("journal").value;

  sauvegarder();
  vibrer();
}

function chargerJournal() {
  const jour = getJour(jourActuel);

  document.getElementById("journal").value =
    jour.journal || "";
}


/* =========================================================
   OBJECTIF
   ========================================================= */

function modifierObjectif() {
  const ancien = donnees.objectif || "";

  const nouvelObjectif = window.prompt(
    "Quel est ton objectif personnel pour ces 30 jours ?",
    ancien
  );

  if (nouvelObjectif === null) {
    return;
  }

  donnees.objectif = nouvelObjectif.trim();

  sauvegarder();
  chargerObjectif();
}

function chargerObjectif() {
  const element = document.getElementById("objectifTexte");

  if (!element) return;

  element.textContent =
    donnees.objectif ||
    "Définis ton objectif personnel.";
}


/* =========================================================
   TRAITEMENT
   ========================================================= */

function sauverTraitement() {
  const nom = document.getElementById("traitementNom").value.trim();
  const heure = document.getElementById("traitementHeure").value;

  donnees.traitement = {
    nom,
    heure
  };

  sauvegarder();
  chargerTraitement();
  vibrer();
}

function chargerTraitement() {
  const nomInput = document.getElementById("traitementNom");
  const heureInput = document.getElementById("traitementHeure");
  const affichage = document.getElementById("traitementAffichage");

  if (!nomInput || !heureInput || !affichage) return;

  nomInput.value = donnees.traitement?.nom || "";
  heureInput.value = donnees.traitement?.heure || "";

  if (
    donnees.traitement?.nom &&
    donnees.traitement?.heure
  ) {
    affichage.textContent =
      `⏰ Rappel : ${donnees.traitement.nom} à ${donnees.traitement.heure}`;
  } else if (donnees.traitement?.nom) {
    affichage.textContent =
      `💊 Traitement enregistré : ${donnees.traitement.nom}`;
  } else {
    affichage.textContent =
      "Aucun rappel enregistré.";
  }
}


/* =========================================================
   PARAMÈTRES
   ========================================================= */

function ouvrirParametres() {
  const modal = document.getElementById("parametres");

  if (!modal) return;

  modal.classList.remove("hidden");
}

function fermerParametres() {
  const modal = document.getElementById("parametres");

  if (!modal) return;

  modal.classList.add("hidden");
}

function sauverParametres() {
  const vibration = document.getElementById("vibration");

  donnees.vibration = vibration.checked;

  sauvegarder();
}

function chargerParametres() {
  const vibration = document.getElementById("vibration");

  if (vibration) {
    vibration.checked = donnees.vibration !== false;
  }
}


/* =========================================================
   THÈME
   ========================================================= */

function changerTheme() {
  donnees.theme =
    donnees.theme === "dark"
      ? "light"
      : "dark";

  appliquerTheme();
  sauvegarder();
}

function appliquerTheme() {
  document.body.classList.toggle(
    "dark",
    donnees.theme === "dark"
  );

  const bouton = document.getElementById("themeBtn");

  if (bouton) {
    bouton.textContent =
      donnees.theme === "dark"
        ? "☀️"
        : "🌙";
  }
}


/* =========================================================
   LANGUE
   ========================================================= */

function changerLangue(langue) {
  if (!["fr", "en", "es"].includes(langue)) {
    langue = "fr";
  }

  donnees.langue = langue;

  sauvegarder();
  appliquerLangue();
}

function appliquerLangue() {
  const langueSelect = document.getElementById("langue");

  if (langueSelect) {
    langueSelect.value = donnees.langue;
  }

  const t = traductions[donnees.langue];

  const titres = document.querySelectorAll(".section-title h2");

  if (titres.length >= 7) {
    titres[0].textContent = "Défi du jour";
    titres[1].textContent = t.objective;
    titres[2].textContent = t.calendar;
    titres[3].textContent = t.habits;
    titres[4].textContent = t.wellbeing;
    titres[5].textContent = t.meals;
    titres[6].textContent = t.movement;
  }

  document.getElementById("titreJour").textContent =
    `${t.day} ${jourActuel}`;

  afficherDate();
  afficherDefi();
  afficherHabitudes();
  afficherBadges();
}


/* =========================================================
   SAISON
   ========================================================= */

function saisonAutomatique() {
  const mois = new Date().getMonth() + 1;

  if (mois >= 3 && mois <= 5) {
    return "printemps";
  }

  if (mois >= 6 && mois <= 8) {
    return "ete";
  }

  if (mois >= 9 && mois <= 11) {
    return "automne";
  }

  return "hiver";
}

function changerSaison(saison) {
  donnees.saison = saison;

  sauvegarder();
  creerDecorations();
}

function creerDecorations() {
  const container = document.getElementById("decorations");

  if (!container) return;

  let saison = donnees.saison;

  if (saison === "auto") {
    saison = saisonAutomatique();
  }

  const emojis = decorations[saison] || decorations.printemps;

  container.innerHTML = "";

  emojis.forEach(emoji => {
    const span = document.createElement("span");
    span.textContent = emoji;
    container.appendChild(span);
  });

  const select = document.getElementById("saison");

  if (select) {
    select.value = donnees.saison;
  }
}


/* =========================================================
   VIBRATION
   ========================================================= */

function vibrer() {
  if (
    donnees.vibration !== false &&
    "vibrate" in navigator
  ) {
    try {
      navigator.vibrate(30);
    } catch (erreur) {
      // Certaines plateformes bloquent la vibration.
    }
  }
}


/* =========================================================
   SURPRISE
   ========================================================= */

function surprise() {
  const element = document.getElementById("surpriseTexte");

  if (!element) return;

  const index = Math.floor(
    Math.random() * surprises.length
  );

  element.textContent = surprises[index];

  vibrer();
}


/* =========================================================
   MUSIQUE
   ========================================================= */

function toggleMusique() {
  if (musiqueActive) {
    arreterMusique();
  } else {
    lancerMusique();
  }
}

function lancerMusique() {
  try {
    const AudioContext =
      window.AudioContext ||
      window.webkitAudioContext;

    if (!AudioContext) {
      alert("La musique n’est pas disponible sur ce navigateur.");
      return;
    }

    audioContext = new AudioContext();

    gainNode = audioContext.createGain();
    gainNode.gain.value = 0.035;
    gainNode.connect(audioContext.destination);

    oscillateur = audioContext.createOscillator();

    oscillateur.type = "sine";
    oscillateur.frequency.value = 220;

    oscillateur.connect(gainNode);
    oscillateur.start();

    musiqueActive = true;

    document.getElementById("musicBtn").textContent =
      "⏹️ Arrêter la musique";

  } catch (erreur) {
    console.error("Erreur musique :", erreur);
  }
}

function arreterMusique() {
  try {
    if (oscillateur) {
      oscillateur.stop();
      oscillateur.disconnect();
      oscillateur = null;
    }

    if (audioContext) {
      audioContext.close();
      audioContext = null;
    }

  } catch (erreur) {
    console.error("Erreur arrêt musique :", erreur);
  }

  musiqueActive = false;

  const bouton = document.getElementById("musicBtn");

  if (bouton) {
    bouton.textContent = "🎵 Lancer la musique";
  }
}


/* =========================================================
   FERMETURE DU MODAL EN CLIQUANT À CÔTÉ
   ========================================================= */

document.addEventListener("click", function(event) {
  const modal = document.getElementById("parametres");

  if (!modal) return;

  if (
    event.target === modal &&
    !modal.classList.contains("hidden")
  ) {
    fermerParametres();
  }
});


/* =========================================================
   TOUCHE ÉCHAP
   ========================================================= */

document.addEventListener("keydown", function(event) {
  if (event.key === "Escape") {
    fermerParametres();
  }
});


/* =========================================================
   INITIALISATION
   ========================================================= */

document.addEventListener("DOMContentLoaded", function() {

  appliquerTheme();
  chargerParametres();

  const langueSelect = document.getElementById("langue");
  if (langueSelect) {
    langueSelect.value = donnees.langue;
  }

  const saisonSelect = document.getElementById("saison");
  if (saisonSelect) {
    saisonSelect.value = donnees.saison;
  }

  creerDecorations();
  chargerObjectif();
  chargerTraitement();

  afficherAccueil();

});
