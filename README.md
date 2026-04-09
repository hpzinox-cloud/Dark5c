<!DOCTYPE html>
<html lang="fr">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1.0"/>
<title>Dark 5C - Fichiers & Photos</title>
<!-- Vos styles ici -->
<style>
  *, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }
  body {
    font-family: 'Share Tech Mono', monospace;
    background: #050d05;
    color: #00ff41;
    min-height: 100vh;
    overflow-x: hidden;
    background-image: url('1775724285910_image.png'); /* votre image de fond */
    background-size: cover;
    background-position: center top;
    filter: brightness(0.15) saturate(0.3) hue-rotate(100deg);
    position: relative;
  }
  .overlay {
    position: fixed; top:0; left:0; width:100%; height:100%; background: repeating-linear-gradient(0deg, transparent, transparent 2px, rgba(0,0,0,0.07) 2px, rgba(0,0,0,0.07) 4px); z-index:1; pointer-events:none;
  }
  .container {
    position: relative; z-index:2; max-width: 960px; margin: 0 auto; padding: 20px;
  }
  h1 {
    text-align: center; font-family: 'Orbitron', monospace; font-size: calc(32px + 4vw); font-weight: 900; color: var(--g);
    text-shadow: 0 0 20px var(--g), 0 0 60px rgba(0,255,65,0.3); margin-bottom: 20px;
  }
  /* Zone de drag & drop */
  .dropzone {
    border: 2px dashed var(--border);
    border-radius: 8px;
    padding: 40px 20px;
    text-align: center;
    cursor: pointer;
    margin-bottom: 20px;
    background: rgba(0,255,65,0.05);
    transition: background 0.3s, border-color 0.3s;
  }
  .dropzone:hover {
    border-color: var(--g);
    background: rgba(0,255,65,0.1);
  }
  /* Bouton pour ouvrir la sélection */
  #fileInput {
    display: none;
  }
  /* Liste des fichiers */
  #fileList {
    max-height: 400px;
    overflow-y: auto;
    display: flex;
    flex-wrap: wrap;
    gap: 10px;
    margin-top: 20px;
  }
  .file-item {
    background: rgba(0, 20, 5, 0.88);
    border: 1px solid var(--border);
    border-radius: 8px;
    padding: 10px;
    width: calc(33% - 20px);
    box-shadow: 0 0 10px rgba(0,255,65,0.2);
    position: relative;
  }
  .file-item img {
    max-width: 100%;
    border-radius: 4px;
  }
  .file-name {
    word-break: break-all;
    font-size: 12px;
    margin-top: 8px;
    color: #fff;
  }
  /* Variables */
:root {
  --g: #00ff41;
  --gd: #00aa2a;
  --border: rgba(0,255,65,0.25);
}
</style>
</head>
<body>
<div class="overlay"></div>
<div class="container">
  <h1>Dark 5C - Fichiers & Photos</h1>
  <!-- Zone de drag & drop -->
  <div class="dropzone" id="dropzone">
    Glissez-déposez des fichiers ici ou cliquez pour sélectionner
  </div>
  <button id="selectBtn" style="margin-top:10px; padding:10px 20px; background:#00ff41; border:none; border-radius:6px; cursor:pointer;">Sélectionner des fichiers</button>
  <input type="file" id="fileInput" multiple />
  
  <!-- Liste des fichiers -->
  <div id="fileList"></div>
</div>

<script>
  const dropzone = document.getElementById('dropzone');
  const fileInput = document.getElementById('fileInput');
  const selectBtn = document.getElementById('selectBtn');
  const fileList = document.getElementById('fileList');

  // Charger fichiers depuis localStorage si existants
  let filesData = JSON.parse(localStorage.getItem('files')) || [];

  function saveFiles() {
    localStorage.setItem('files', JSON.stringify(filesData));
  }

  function displayFiles() {
    fileList.innerHTML = '';
    filesData.forEach((file, index) => {
      const item = document.createElement('div');
      item.className = 'file-item';
      if (file.type.startsWith('image/')) {
        const img = document.createElement('img');
        img.src = file.dataUrl;
        item.appendChild(img);
      } else {
        // Affichage simple pour autres fichiers
        const link = document.createElement('a');
        link.href = file.dataUrl;
        link.target = '_blank';
        link.textContent = file.name;
        link.style.color = '#fff';
        link.style.textDecoration = 'underline';
        item.appendChild(link);
      }
      // Nom du fichier
      const nameDiv = document.createElement('div');
      nameDiv.className = 'file-name';
      nameDiv.textContent = file.name;
      item.appendChild(nameDiv);
      // Bouton supprimer
      const delBtn = document.createElement('button');
      delBtn.textContent = 'Supprimer';
      delBtn.style.position='absolute'; top:'5px'; right:'5px'; font-size:'10px'; padding:'2px 4px'; border='none'; border-radius='4px'; background='#ff3333'; cursor='pointer';
      delBtn.onclick = () => {
        filesData.splice(index,1);
        saveFiles();
        displayFiles();
      };
      item.appendChild(delBtn);

      fileList.appendChild(item);
    });
  }

  displayFiles();

  // Fonction pour traiter les fichiers
  function handleFiles(files) {
    Array.from(files).forEach(file => {
      const reader = new FileReader();
      reader.onload = () => {
        filesData.push({
          name: file.name,
          dataUrl: reader.result,
          type: file.type
        });
        saveFiles();
        displayFiles();
      };
      reader.readAsDataURL(file);
    });
  }

  // Drag & Drop
  dropzone.addEventListener('dragover', (e) => {
    e.preventDefault();
    dropzone.style.background = 'rgba(0,255,65,0.2)';
  });
  dropzone.addEventListener('dragleave', () => {
    dropzone.style.background = 'rgba(0,255,65,0.05)';
  });
  dropzone.addEventListener('drop', (e) => {
    e.preventDefault();
    dropzone.style.background = 'rgba(0,255,65,0.05)';
    handleFiles(e.dataTransfer.files);
  });

  // Cliquer pour ouvrir la sélection
  dropzone.addEventListener('click', () => {
    fileInput.click();
  });
  selectBtn.addEventListener('click', () => {
    fileInput.click();
  });

  // Sélectionner des fichiers
  fileInput.addEventListener('change', () => {
    handleFiles(fileInput.files);
    fileInput.value = ''; // Reset
  });
</script>
</body>
</html>
