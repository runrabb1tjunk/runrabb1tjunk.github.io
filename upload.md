---
layout: page
permalink: /upload/
---

<script src="https://cdnjs.cloudflare.com/ajax/libs/ethers/6.13.2/ethers.umd.min.js"></script>

<div class="web3-gate-wrapper">
  <style>
    .upload-card {
      background-image: url('/assets/background.webp') !important;
      background-repeat: repeat !important;
      border: 1px solid #ff4d4d;
      border-radius: 8px;
      padding: 25px;
      color: #fff;
      max-width: 800px;
      margin: 20px auto;
      box-sizing: border-box;
    }
    .upload-dropzone {
      border: 2px dashed #ff4d4d;
      border-radius: 6px;
      padding: 30px;
      text-align: center;
      cursor: pointer;
      background: rgba(0,0,0,0.2);
      margin-bottom: 20px;
      transition: background 0.2s ease;
    }
    .upload-dropzone.drag-over {
      background: rgba(255, 77, 77, 0.2);
    }
    .image-preview-card {
      background: rgba(0,0,0,0.4);
      border: 1px solid rgba(255,255,255,0.2);
      border-radius: 6px;
      padding: 15px;
      margin-bottom: 15px;
      display: flex;
      gap: 15px;
      align-items: center;
    }
    .image-preview-card img {
      width: 100px;
      height: 100px;
      object-fit: cover;
      border-radius: 4px;
    }
    .form-fields {
      display: flex;
      flex-direction: column;
      gap: 10px;
      flex-grow: 1;
    }
    .dimensions-row {
      display: flex;
      gap: 10px;
      align-items: center;
    }
    .form-fields input {
      background: rgba(0,0,0,0.5);
      border: 1px solid #ff4d4d;
      color: #fff;
      padding: 8px;
      border-radius: 4px;
      font-family: inherit;
    }
    .file-size-tag {
      font-size: 0.8em;
      color: #34d399;
      margin-top: 2px;
    }
    .status-msg {
      margin-top: 15px;
      font-weight: bold;
      color: #34d399;
      text-align: center;
    }

    .existing-nfts-section {
      margin-top: 35px;
      border-top: 1px dashed rgba(255, 77, 77, 0.4);
      padding-top: 20px;
    }
    .existing-grid {
      display: grid;
      grid-template-columns: repeat(auto-fill, minmax(180px, 1fr));
      gap: 15px;
      margin-top: 15px;
    }
    .nft-item-card {
      background: rgba(0,0,0,0.5);
      border: 1px solid rgba(255, 77, 77, 0.4);
      border-radius: 6px;
      padding: 10px;
      text-align: center;
      position: relative;
    }
    .nft-item-card.minted-on-chain {
      border: 2px solid #34d399 !important;
      background: rgba(16, 185, 129, 0.1) !important;
    }
    .nft-item-card img {
      width: 100%;
      height: 140px;
      object-fit: cover;
      border-radius: 4px;
      margin-bottom: 8px;
    }
    .nft-checkbox {
      position: absolute;
      top: 14px;
      left: 14px;
      transform: scale(1.3);
      cursor: pointer;
    }
    .nft-item-card .title {
      font-weight: bold;
      font-size: 0.9em;
      color: #fff;
      white-space: nowrap;
      overflow: hidden;
      text-overflow: ellipsis;
    }
    .nft-item-card .dims {
      font-size: 0.8em;
      color: #aaa;
      margin-top: 4px;
    }
    .mint-badge {
      font-size: 0.75em;
      background: #34d399;
      color: #000;
      padding: 2px 6px;
      border-radius: 4px;
      font-weight: bold;
      margin-top: 4px;
      display: inline-block;
    }
    .mint-price-input {
      margin-top: 6px;
      width: 90%;
      background: rgba(0,0,0,0.7);
      border: 1px solid #34d399;
      color: #fff;
      padding: 4px;
      border-radius: 4px;
      font-size: 0.85em;
      text-align: center;
    }
    .action-buttons-row {
      display: flex;
      gap: 10px;
      margin-top: 15px;
      flex-wrap: wrap;
    }
  </style>

  <div class="upload-card">
    <div style="text-align:center; margin-bottom: 15px; font-size: 0.9em; color: #aaa;">
      Download current version of gallery client at 
      <a href="https://demidmorozov.itch.io/sphere-gallery" target="_blank" style="color: #ff4d4d; font-weight: bold; text-decoration: underline;">
        https://demidmorozov.itch.io/sphere-gallery
      </a>
    </div>

    <div id="upload-auth-status" class="status-lock-notice" style="text-align:center; margin-bottom:15px;">
      Log in and connect wallet to upload
    </div>

    <div id="upload-main-area" class="hidden">
      <!-- Dropzone -->
      <div class="upload-dropzone" id="dropzone">
        <i class="bi bi-cloud-arrow-up" style="font-size:2rem; color:#ff4d4d;"></i>
        <p>Click or drag new images here (Batch Upload & Auto-Compress to &lt;300 KB)</p>
        <input type="file" id="file-input" multiple accept="image/*" class="hidden">
      </div>

      <!-- Item Preview List -->
      <div id="preview-list"></div>

      <button id="btn-submit-pinata" class="btn-submit" style="display:none; width: 100%; padding: 12px; margin-top: 15px; background: #ff4d4d; color: #fff; border: none; border-radius: 4px; font-weight: bold; cursor: pointer;">
        Upload
      </button>

      <div id="upload-loader" class="status-msg"></div>

      <!-- Раздел с уже загруженными работами -->
      <div class="existing-nfts-section">
        <h3 style="color:#ff4d4d; margin-bottom: 5px;">Your Uploaded Works (IPFS)</h3>
        <div id="existing-nfts-status" style="font-size: 0.85em; color: #aaa;">Loading existing works...</div>
        <div id="existing-nfts-grid" class="existing-grid"></div>
        
        <div class="action-buttons-row">
          <button id="btn-mint-selected" style="display:none; padding: 8px 16px; background: #10b981; color: #fff; border: none; border-radius: 4px; font-weight: bold; cursor: pointer;" onclick="handleMintButtonAction()">
            Mint to chain
          </button>
          <button id="btn-delete-selected" style="display:none; padding: 8px 16px; background: #b91c1c; color: #fff; border: none; border-radius: 4px; font-weight: bold; cursor: pointer;" onclick="deleteSelectedExistingNFTs()">
            Delete selected
          </button>
        </div>
      </div>
    </div>
  </div>
</div>

<script>
let processedImages = [];
window.lastKnownFolderCid = null;
let isFetchingNFTs = false;

// Генератор случайного хэша
function generateRandomHash() {
    const array = new Uint8Array(8);
    crypto.getRandomValues(array);
    const hash = Array.from(array, byte => byte.toString(16).padStart(2, '0')).join('');
    console.log("🛠 [DEBUG] Generated random hash:", hash);
    return hash;
}

(function() {
    const WORKER_URL = "https://throbbing-silence-679f.fgfg57468.workers.dev";
    const PINATA_GATEWAY = "https://black-obliged-cricket-162.mypinata.cloud/ipfs/";
    const FACTORY_CONTRACT_ADDRESS = '0x10f44a41BE5CE208d47524Ff6FB77E21a9D8cA52';
    const RPC_URL = 'https://polygon-mainnet.infura.io/v3/0594ea55d3cc4d2ba85c685603762bf6';

    const FACTORY_ABI = [
        "function batchMintAndList(string[] calldata _names, string[] calldata _tokenURIs, uint256[] calldata _prices) external",
        "function walletCreatorTokensForSale(address _creator) external view returns (uint256[] memory)",
        "function getNFTDetails(uint256 tokenId) external view returns (string memory nftName, string memory uri, uint256 price, address creator, address owner, bool isForSale)",
        "event NFTMinted(uint256 indexed tokenId, address indexed creator, string name, uint256 price, string tokenURI)"
    ];

    function updateAuthUI(username, wallet) {
        const statusDiv = document.getElementById("upload-auth-status");
        const mainArea = document.getElementById("upload-main-area");
        if (!statusDiv || !mainArea) return;

        if (username && wallet) {
            statusDiv.className = "status-unlock-notice";
            statusDiv.innerText = `Author: ${username} | Wallet: ${wallet.substring(0,6)}...${wallet.substring(wallet.length-4)}`;
            mainArea.classList.remove("hidden");
            loadExistingNFTs(username, wallet);
        } else {
            statusDiv.className = "status-lock-notice";
            statusDiv.innerText = "Please log in and connect your wallet in the top menu.";
            mainArea.classList.add("hidden");
        }
    }

    function checkAuth() {
        const username = localStorage.getItem('app_currentUser') || localStorage.getItem('currentUser');
        const wallet = localStorage.getItem('app_linkedWallet') || localStorage.getItem('linkedWallet');
        updateAuthUI(username, wallet);
    }

    if (document.readyState === "loading") {
        document.addEventListener("DOMContentLoaded", checkAuth);
    } else {
        checkAuth();
    }

    window.addEventListener("user_auth_status", (e) => {
        const detail = e.detail || {};
        const username = detail.username || localStorage.getItem('app_currentUser') || localStorage.getItem('currentUser');
        const wallet = detail.wallet || localStorage.getItem('app_linkedWallet') || localStorage.getItem('linkedWallet');
        updateAuthUI(username, wallet);
    });

    window.loadExistingNFTs = async function(username, walletAddress) {
        const statusEl = document.getElementById('existing-nfts-status');
        const gridEl = document.getElementById('existing-nfts-grid');
        
        if (!statusEl || !gridEl || isFetchingNFTs) return;

        try {
            isFetchingNFTs = true;
            statusEl.innerText = "Fetching your items from SQL/IPFS & checking blockchain status...";

            const res = await fetch(`${WORKER_URL}/get-user-nfts?username=${encodeURIComponent(username)}&t=${Date.now()}`, {
                cache: 'no-store'
            });
            if (!res.ok) throw new Error("Failed to load user NFTs");

            const data = await res.json();
            gridEl.innerHTML = "";

            if (!data.folderCid || !data.images || data.images.length === 0) {
                statusEl.innerText = "No uploaded items found.";
                window.lastKnownFolderCid = null;
                toggleSubmitButtonVisibility();
                return;
            }

            window.lastKnownFolderCid = data.folderCid;
            let cleanCid = data.folderCid;
            if (cleanCid.includes('/')) {
                cleanCid = cleanCid.substring(0, cleanCid.indexOf('/'));
            }

            let mintedTokensMap = new Map();
            try {
                const provider = new ethers.JsonRpcProvider(RPC_URL);
                const factoryContract = new ethers.Contract(FACTORY_CONTRACT_ADDRESS, FACTORY_ABI, provider);
                const activeIds = await factoryContract.walletCreatorTokensForSale(walletAddress);
                
                for (const tokenId of activeIds) {
                    const details = await factoryContract.getNFTDetails(tokenId);
                    let rawUri = (details.uri || "").trim();
                    let normUri = rawUri.toLowerCase();
                    if (normUri.includes('ipfs/')) normUri = normUri.split('ipfs/').pop();
                    if (normUri.includes('//')) normUri = normUri.split('//').pop();

                    mintedTokensMap.set(normUri, {
                        tokenId: tokenId.toString(),
                        price: ethers.formatUnits(details.price, 6)
                    });
                }
            } catch (chainErr) {
                console.warn("Could not fetch on-chain state:", chainErr);
            }

            const uniqueMap = new Map();
            if (data.images && Array.isArray(data.images)) {
                data.images.forEach(img => {
                    const pureFileName = (img.fileName || '').split('/').pop();
                    const lowerName = pureFileName.toLowerCase();
                    
                    if (!lowerName || lowerName.includes('meta') || lowerName.endsWith('.json')) return;

                    const key = lowerName; 
                    if (!uniqueMap.has(key)) {
                        let isMinted = false;
                        let tokenData = null;

                        for (let [onChainUri, info] of mintedTokensMap.entries()) {
                            if (onChainUri.includes(lowerName) || (cleanCid && onChainUri.includes(cleanCid.toLowerCase()) && onChainUri.includes(lowerName))) {
                                isMinted = true;
                                tokenData = info;
                                break;
                            }
                        }

                        uniqueMap.set(key, {
                            title: img.title ? img.title.trim() : '',
                            fileName: pureFileName,
                            width: img.width,
                            height: img.height,
                            folderCid: cleanCid,
                            url: `${PINATA_GATEWAY}${cleanCid}/${pureFileName}?t=${Date.now()}`,
                            isMinted: isMinted,
                            tokenData: tokenData
                        });
                    }
                });
            }

            const uniqueImages = Array.from(uniqueMap.values());
            statusEl.innerText = `Folder CID: ${cleanCid.substring(0, 12)}... (Total unique works: ${uniqueImages.length})`;

            uniqueImages.forEach((img) => {
                const card = document.createElement('div');
                card.className = "nft-item-card" + (img.isMinted ? " minted-on-chain" : "");
                card.dataset.title = img.title;
                card.dataset.filename = img.fileName;
                card.dataset.width = img.width;
                card.dataset.height = img.height;
                card.dataset.foldercid = img.folderCid;
                card.dataset.isminted = img.isMinted ? "true" : "false";

                let badgeHtml = img.isMinted ? `<div class="mint-badge">✓ Minted (#${img.tokenData.tokenId})</div>` : '';
                let priceInputHtml = img.isMinted ? '' : `<input type="number" step="0.01" min="0.01" value="1.0" class="mint-price-input" placeholder="Price in USDT" title="Set price in USDT">`;

                card.innerHTML = `
                    <input type="checkbox" class="nft-checkbox" onchange="toggleDeleteButton()">
                    <a href="${img.url}" target="_blank">
                        <img src="${img.url}" alt="${img.title}" loading="lazy">
                    </a>
                    <div class="title" title="${img.title}">${img.title}</div>
                    <div class="dims">${img.width} x ${img.height} cm</div>
                    ${badgeHtml}
                    ${priceInputHtml}
                `;
                gridEl.appendChild(card);
            });

            toggleSubmitButtonVisibility();
        } catch (err) {
            statusEl.innerText = "Error loading works: " + err.message;
        } finally {
            isFetchingNFTs = false;
        }
    };

    const dropzone = document.getElementById('dropzone');
    const fileInput = document.getElementById('file-input');

    if (dropzone && fileInput) {
        dropzone.addEventListener('click', () => fileInput.click());
        dropzone.addEventListener('dragover', (e) => { e.preventDefault(); dropzone.classList.add('drag-over'); });
        dropzone.addEventListener('dragleave', (e) => { e.preventDefault(); dropzone.classList.remove('drag-over'); });
        dropzone.addEventListener('drop', (e) => {
            e.preventDefault();
            dropzone.classList.remove('drag-over');
            if (e.dataTransfer && e.dataTransfer.files.length > 0) processFileList(Array.from(e.dataTransfer.files));
        });
        fileInput.addEventListener('change', (e) => processFileList(Array.from(e.target.files)));
    }

    async function compressToMax300KB(canvas) {
        let quality = 0.85;
        let blob = await new Promise(resolve => canvas.toBlob(resolve, 'image/jpeg', quality));
        const maxSize = 300 * 1024;

        while (blob.size > maxSize && quality > 0.2) {
            quality -= 0.1;
            blob = await new Promise(resolve => canvas.toBlob(resolve, 'image/jpeg', quality));
        }

        if (blob.size > maxSize) {
            let scale = 0.9;
            const tempCanvas = document.createElement('canvas');
            const ctx = tempCanvas.getContext('2d');

            while (blob.size > maxSize && scale > 0.2) {
                tempCanvas.width = Math.round(canvas.width * scale);
                tempCanvas.height = Math.round(canvas.height * scale);
                ctx.drawImage(canvas, 0, 0, tempCanvas.width, tempCanvas.height);
                blob = await new Promise(resolve => tempCanvas.toBlob(resolve, 'image/jpeg', 0.7));
                scale -= 0.15;
            }
        }
        return blob;
    }

    function processFileList(files) {
        if (!files.length) return;

        files.forEach(file => {
            if (!file.type.startsWith('image/')) return;

            const reader = new FileReader();
            reader.onload = (e) => {
                const img = new Image();
                img.onload = async () => {
                    const canvas = document.createElement('canvas');
                    canvas.width = img.width;
                    canvas.height = img.height;
                    const ctx = canvas.getContext('2d');
                    ctx.drawImage(img, 0, 0);

                    const blob = await compressToMax300KB(canvas);

                    const widthCm = 50;
                    const heightCm = parseFloat((50 * (img.height / img.width)).toFixed(1));
                    const sizeKb = (blob.size / 1024).toFixed(1);
                    const baseTitle = file.name.replace(/\.[^/.]+$/, "").trim();

                    const item = {
                        id: 'img_' + Math.random().toString(36).substr(2, 9),
                        blob: blob,
                        sizeKb: sizeKb,
                        originalName: file.name,
                        title: baseTitle,
                        widthCm: widthCm,
                        heightCm: heightCm,
                        dataUrl: URL.createObjectURL(blob)
                    };
                    processedImages.push(item);
                    renderPreviews();
                };
                img.src = e.target.result;
            };
            reader.readAsDataURL(file);
        });
    }

    function renderPreviews() {
        const container = document.getElementById('preview-list');
        if (!container) return;
        container.innerHTML = "";

        processedImages.forEach((item, idx) => {
            const card = document.createElement('div');
            card.className = "image-preview-card";
            card.innerHTML = `
                <img src="${item.dataUrl}">
                <div class="form-fields">
                    <label style="color:#ff4d4d; font-size:0.9em;">Title:</label>
                    <input type="text" value="${item.title}" data-idx="${idx}" class="input-title">
                    <div class="dimensions-row">
                        <div>
                            <label style="color:#888; font-size:0.8em;">Width (cm):</label>
                            <input type="number" step="0.1" value="${item.widthCm}" style="width: 100px;" data-idx="${idx}" class="input-w">
                        </div>
                        <div>
                            <label style="color:#888; font-size:0.8em;">Height (cm):</label>
                            <input type="number" step="0.1" value="${item.heightCm}" style="width: 100px;" data-idx="${idx}" class="input-h">
                        </div>
                    </div>
                    <div class="file-size-tag">Size: ~${item.sizeKb} KB (JPEG)</div>
                </div>
                <button data-idx="${idx}" class="btn-remove" style="background:none; border:none; color:#ff4d4d; cursor:pointer;"><i class="bi bi-trash"></i></button>
            `;
            container.appendChild(card);
        });

        container.querySelectorAll('.input-title').forEach(el => {
            el.addEventListener('change', (e) => {
                processedImages[e.target.dataset.idx].title = e.target.value.trim();
            });
        });
        container.querySelectorAll('.input-w').forEach(el => {
            el.addEventListener('change', (e) => processedImages[e.target.dataset.idx].widthCm = parseFloat(e.target.value));
        });
        container.querySelectorAll('.input-h').forEach(el => {
            el.addEventListener('change', (e) => processedImages[e.target.dataset.idx].heightCm = parseFloat(e.target.value));
        });
        container.querySelectorAll('.btn-remove').forEach(el => {
            el.addEventListener('click', (e) => {
                processedImages.splice(e.currentTarget.dataset.idx, 1);
                renderPreviews();
            });
        });

        toggleSubmitButtonVisibility();
    }

    function toggleSubmitButtonVisibility() {
        const existingCards = document.querySelectorAll('.nft-item-card');
        const btn = document.getElementById('btn-submit-pinata');
        if (btn) {
            btn.style.display = (processedImages.length > 0 || existingCards.length > 0) ? 'block' : 'none';
        }
    }

    window.toggleDeleteButton = function() {
        const checkboxes = document.querySelectorAll('.nft-checkbox:checked');
        const deleteBtn = document.getElementById('btn-delete-selected');
        const mintBtn = document.getElementById('btn-mint-selected');
        const displayStyle = checkboxes.length > 0 ? 'inline-block' : 'none';
        if (deleteBtn) deleteBtn.style.display = displayStyle;
        if (mintBtn) mintBtn.style.display = displayStyle;
    };

    window.deleteSelectedExistingNFTs = async function() {
        const checkboxes = document.querySelectorAll('.nft-checkbox:checked');
        if (!checkboxes.length) return;
        if (!confirm("Delete selected images? Changes will be saved to IPFS.")) return;

        checkboxes.forEach(cb => {
            const card = cb.closest('.nft-item-card');
            if (card) card.remove();
        });

        window.toggleDeleteButton();
        await processAndUploadDirect();
    };

    window.handleMintButtonAction = async function() {
        const checkboxes = document.querySelectorAll('.nft-checkbox:checked');
        if (!checkboxes.length) return;

        if (typeof ethers === 'undefined' || !window.ethereum) {
            alert("Ethers library or Web3 wallet not detected.");
            return;
        }

        const loader = document.getElementById('upload-loader');
        loader.innerText = "Connecting to wallet and checking account...";

        try {
            const provider = new ethers.BrowserProvider(window.ethereum);
            await provider.send("eth_requestAccounts", []);
            const signer = await provider.getSigner();
            const connectedWallet = (await signer.getAddress()).toLowerCase();
            const expectedWallet = (localStorage.getItem('app_linkedWallet') || localStorage.getItem('linkedWallet') || "").toLowerCase();

            if (expectedWallet && connectedWallet !== expectedWallet) {
                alert(`Error: Connected wallet does not match your linked account.`);
                loader.innerText = "";
                return;
            }

            loader.innerText = "Preparing mint transaction...";
            const factoryContract = new ethers.Contract(FACTORY_CONTRACT_ADDRESS, FACTORY_ABI, signer);

            const namesArray = [];
            const urisArray = [];
            const pricesArray = [];

            for (let cb of checkboxes) {
                const card = cb.closest('.nft-item-card');
                if (card) {
                    if (card.dataset.isminted === "true") {
                        alert("One of the selected items is already minted on-chain!");
                        loader.innerText = "";
                        return;
                    }

                    const title = card.dataset.title || "NFTAsset";
                    const imgEl = card.querySelector('img');
                    let imgUrl = imgEl ? imgEl.src.split('?')[0].replace('https://', '') : '';
                    const priceVal = card.querySelector('.mint-price-input')?.value.trim() || "1.0";

                    namesArray.push(title);
                    urisArray.push(imgUrl);
                    pricesArray.push(ethers.parseUnits(priceVal, 6));
                }
            }

            const estimatedGas = await factoryContract.batchMintAndList.estimateGas(namesArray, urisArray, pricesArray);
            const tx = await factoryContract.batchMintAndList(namesArray, urisArray, pricesArray, {
                gasLimit: (estimatedGas * 140n) / 100n
            });

            loader.innerText = `Transaction sent! Waiting for confirmation...`;
            await tx.wait();

            loader.innerText = `✅ Successfully minted ${namesArray.length} NFT(s)!`;
            alert(`Successfully minted ${namesArray.length} NFT(s)!`);

            loadExistingNFTs(localStorage.getItem('app_currentUser') || localStorage.getItem('currentUser'), localStorage.getItem('app_linkedWallet') || localStorage.getItem('linkedWallet'));

        } catch (err) {
            console.error(err);
            loader.innerText = `Mint error: ${err.message}`;
        }
    };

    async function processAndUploadDirect() {
        const loader = document.getElementById('upload-loader');
        const username = localStorage.getItem('app_currentUser') || localStorage.getItem('currentUser');
        const wallet = localStorage.getItem('app_linkedWallet') || localStorage.getItem('linkedWallet');

        if (!wallet || !username) {
            alert("Authorization missing.");
            return;
        }

        console.log("🚀 [DEBUG] Starting upload process for user:", username, "wallet:", wallet);
        loader.innerText = "Forming package and uploading to server...";

        const formData = new FormData();
        formData.append("username", username);
        formData.append("wallet", wallet);

        const newImagesMeta = processedImages.map((item, index) => {
            const randomHash = generateRandomHash();
            const fileName = `${randomHash}.jpg`;
            
            console.log(`📸 [DEBUG] Image [${index}] original title: "${item.title}" -> assigned hash filename: "${fileName}"`);
            formData.append(`file_${index}`, item.blob, fileName);

            return {
                title: item.title,
                fileName: fileName,
                width: item.widthCm,
                height: item.heightCm,
                isExisting: false
            };
        });

        const existingCards = document.querySelectorAll('.nft-item-card');
        const retainedExistingMeta = [];
        let targetFolderCid = window.lastKnownFolderCid;

        existingCards.forEach(card => {
            if (card.dataset.foldercid) targetFolderCid = card.dataset.foldercid;
            retainedExistingMeta.push({
                title: card.dataset.title,
                fileName: card.dataset.filename,
                width: parseFloat(card.dataset.width),
                height: parseFloat(card.dataset.height),
                isExisting: true,
                folderCid: card.dataset.foldercid
            });
        });

        const combined = [...newImagesMeta, ...retainedExistingMeta];
        const uniqueMap = new Map();
        combined.forEach(img => {
            const key = (img.fileName || '').toLowerCase().trim();
            if (key && !uniqueMap.has(key)) uniqueMap.set(key, img);
        });

        const finalImagesPayload = Array.from(uniqueMap.values());
        console.log("📦 [DEBUG] Final payload 'images' JSON being sent:", JSON.stringify(finalImagesPayload, null, 2));

        formData.append("images", JSON.stringify(finalImagesPayload));
        if (targetFolderCid) formData.append("oldFolderCid", targetFolderCid);

        try {
            console.log("🌐 [DEBUG] Sending POST request to WORKER_URL:", WORKER_URL);
            const response = await fetch(WORKER_URL, { method: "POST", body: formData });
            const resData = await response.json();
            console.log("📥 [DEBUG] Response from Worker:", resData);

            if (!response.ok) throw new Error(resData.error || "Upload failed");

            loader.innerText = `Successfully saved and updated in Pinata!`;
            processedImages = [];
            renderPreviews();
            loadExistingNFTs(username, wallet);

        } catch (err) {
            console.error("❌ [DEBUG] Upload error:", err);
            loader.innerText = `Error: ${err.message}`;
        }
    }

    document.getElementById('btn-submit-pinata')?.addEventListener('click', processAndUploadDirect);
})();
</script>