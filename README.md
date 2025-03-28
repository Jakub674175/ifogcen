# ifogcen

<!DOCTYPE html>
<html lang="pl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>iFOGcen</title>
    <style>
        body {
            background-color: black;
            color: white;
            font-family: Arial, sans-serif;
            text-align: center;
            padding: 20px;
        }
        h1, h2 {
            margin-bottom: 20px;
        }
        #nicknameInput, #postInput {
            width: 70%;
            padding: 10px;
            background: black;
            color: white;
            border: 1px solid black;
            font-size: 16px;
            margin-bottom: 20px;
        }
        #postContainerWrapper {
            margin-top: 20px;
            height: 800px;
            overflow-y: auto;
            border: 1px solid black;
            padding: 10px;
            display: flex;
            flex-direction: column-reverse;
            background: black;
        }
        #postContainer {
            display: flex;
            flex-direction: column;
        }
        .post {
            border-bottom: 1px solid gray;
            padding: 10px 0;
        }
        .nickname {
            font-weight: bold;
            color: lightblue;
        }
        .timestamp {
            color: gray;
            font-size: 0.8em;
        }
        .input-container {
            display: flex;
            justify-content: center;
            align-items: center;
            gap: 10px;
        }
        #resetButton {
            background-color: white;
            color: black;
            border: none;
            padding: 5px 10px;
            cursor: pointer;
            font-size: 14px;
            border-radius: 5px;
        }
    </style>
</head>
<body>
    <h1>iFOGcen</h1>
    <h2>NA DZIŚ</h2>
    <h4>LORD KRUSZWIL</h4>
    <p>kanał PowrotSquadu ma już 44.1k subów</p>
    
    <input type="text" id="nicknameInput" placeholder="Podaj swój nick..." onkeypress="setNickname(event)">
    
    <div id="postSection" style="display:none;">
        <div class="input-container">
            <input type="text" id="postInput" placeholder="Napisz coś..." onkeypress="handleKeyPress(event)">
            <button id="resetButton" onclick="resetNickname()">Nowy nick</button>
        </div>
        <div id="postContainerWrapper">
            <div id="postContainer"></div>
        </div>
    </div>
    
    <script>
        let nickname = "Anonim";
        let activeNicknames = JSON.parse(localStorage.getItem("activeNicknames")) || {};

        function setNickname(event) {
            if (event.key === "Enter") {
                let input = document.getElementById("nicknameInput").value.trim();
                
                if (input.toLowerCase() === "admin") {
                    let passcode = prompt("Podaj kod dostępu:");
                    if (passcode !== "Jak18Uplifter!") {
                        alert("Nick jest zablokowany!");
                        return;
                    }
                }

                if (!input) input = "Anonim";

                // Sprawdzenie, czy nick jest aktywny lub zablokowany
                if (activeNicknames[input] && activeNicknames[input] > Date.now()) {
                    alert("Ten nick jest już zajęty!");
                    return;
                }

                nickname = input;
                activeNicknames[nickname] = null;
                localStorage.setItem("activeNicknames", JSON.stringify(activeNicknames));

                document.getElementById("nicknameInput").style.display = "none";
                document.getElementById("postSection").style.display = "block";
            }
        }

        function handleKeyPress(event) {
            if (event.key === "Enter") {
                addPost();
            }
        }

        function addPost() {
            let input = document.getElementById("postInput");
            let text = input.value.trim();
            if (text === "") return;

            let postContainer = document.getElementById("postContainer");
            let postElement = document.createElement("div");
            postElement.classList.add("post");

            let timestamp = new Date().toLocaleString();
            postElement.innerHTML = `<p class="nickname">${nickname}</p><p>${text} <span class="timestamp">(${timestamp})</span></p>`;

            postContainer.appendChild(postElement);
            input.value = "";
            
            savePosts();
            scrollToBottom();
        }

        function savePosts() {
            let posts = [];
            let postElements = document.getElementsByClassName("post");
            for (let post of postElements) {
                let nickname = post.querySelector(".nickname").textContent;
                let text = post.querySelector("p").textContent;
                posts.push({ nickname, text });
            }
            localStorage.setItem("posts", JSON.stringify(posts));
        }

        function loadPosts() {
            let savedPosts = localStorage.getItem("posts");
            if (savedPosts) {
                let posts = JSON.parse(savedPosts);
                for (let post of posts) {
                    let postContainer = document.getElementById("postContainer");
                    let postElement = document.createElement("div");
                    postElement.classList.add("post");
                    let timestamp = new Date().toLocaleString();
                    postElement.innerHTML = `<p class="nickname">${post.nickname}</p><p>${post.text} <span class="timestamp">(${timestamp})</span></p>`;
                    postContainer.appendChild(postElement);
                }
            }
        }

        function clearPostsAtOneAM() {
            let now = new Date();
            if (now.getHours() === 1 && now.getMinutes() === 0) {
                localStorage.removeItem("posts");
                document.getElementById("postContainer").innerHTML = '';
            }
        }

        function resetNickname() {
            activeNicknames[nickname] = Date.now() + 60 * 60 * 1000; // Nick zablokowany na 1 godzinę
            localStorage.setItem("activeNicknames", JSON.stringify(activeNicknames));
            
            document.getElementById("nicknameInput").style.display = "block";
            document.getElementById("nicknameInput").value = "";
            document.getElementById("postSection").style.display = "none";
            nickname = "Anonim";
        }

        function scrollToBottom() {
            let postContainerWrapper = document.getElementById("postContainerWrapper");
            postContainerWrapper.scrollTop = postContainerWrapper.scrollHeight;
        }

        document.getElementById("postContainer").addEventListener("contextmenu", function(event) {
            event.preventDefault();
            
            if (nickname.toLowerCase() !== "admin") {
                alert("Tylko administrator może usuwać wpisy!");
                return;
            }

            let targetPost = event.target.closest(".post");
            if (targetPost) {
                if (confirm("Czy na pewno chcesz usunąć ten wpis?")) {
                    targetPost.remove();
                    savePosts();
                }
            }
        });

        window.onload = function() {
            loadPosts();
            setInterval(clearPostsAtOneAM, 60000);
        };
    </script>
</body>
</html>
