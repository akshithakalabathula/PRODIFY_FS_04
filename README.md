<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Social Media App</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            margin: 0;
            padding: 0;
            background-color: #f0f2f5;
        }
        .container {
            width: 50%;
            margin: 20px auto;
            background: white;
            padding: 20px;
            box-shadow: 0px 0px 10px rgba(0, 0, 0, 0.1);
            border-radius: 10px;
        }
        h2 {
            text-align: center;
        }
        input, textarea {
            width: 100%;
            padding: 10px;
            margin: 10px 0;
            border: 1px solid #ccc;
            border-radius: 5px;
        }
        button {
            background: #1877f2;
            color: white;
            border: none;
            padding: 10px;
            cursor: pointer;
            width: 100%;
            border-radius: 5px;
        }
        button:hover {
            background: #0e5ac7;
        }
        .post {
            background: white;
            padding: 15px;
            margin-top: 15px;
            border-radius: 5px;
            box-shadow: 0px 0px 5px rgba(0, 0, 0, 0.1);
        }
        .post img, .post video {
            max-width: 100%;
            border-radius: 5px;
            margin-top: 10px;
        }
        .like-btn, .follow-btn {
            color: red;
            cursor: pointer;
            margin-right: 10px;
        }
        .follow-btn {
            color: blue;
        }
        .comment-section {
            margin-top: 10px;
        }
        .comment {
            background: #f0f0f0;
            padding: 5px;
            border-radius: 5px;
            margin-top: 5px;
        }
        .notifications {
            background: #fffae5;
            padding: 10px;
            margin-top: 10px;
            border-radius: 5px;
        }
        .trending {
            background: #e3f2fd;
            padding: 10px;
            margin-top: 20px;
            border-radius: 5px;
        }
    </style>
</head>
<body>

<div class="container">
    <h2>Create Profile</h2>
    <input type="text" id="username" placeholder="Enter your name">
    <button onclick="createProfile()">Create Profile</button>
</div>

<div class="container" id="postSection" style="display:none;">
    <h2>Create a Post</h2>
    <textarea id="postContent" placeholder="Write something..."></textarea>
    <input type="file" id="mediaUpload">
    <input type="text" id="tagUsers" placeholder="Tag users (comma separated)">
    <button onclick="createPost()">Post</button>
</div>

<div class="container">
    <h2>Feed</h2>
    <div id="feed"></div>
</div>

<div class="container trending">
    <h2>🔥 Trending</h2>
    <div id="trending"></div>
</div>

<div class="container notifications">
    <h2>🔔 Notifications</h2>
    <div id="notifications"></div>
</div>

<script>
    let currentUser = null;
    let users = {};
    let posts = [];

    function createProfile() {
        let username = document.getElementById("username").value;
        if (username.trim() === "") {
            alert("Please enter your name");
            return;
        }
        currentUser = username;
        if (!users[currentUser]) {
            users[currentUser] = { following: [], followers: [] };
        }
        document.getElementById("postSection").style.display = "block";
        alert("Profile created successfully!");
    }

    function createPost() {
        if (!currentUser) {
            alert("Create a profile first!");
            return;
        }

        let postContent = document.getElementById("postContent").value;
        let mediaFile = document.getElementById("mediaUpload").files[0];
        let taggedUsers = document.getElementById("tagUsers").value.split(",").map(u => u.trim()).filter(Boolean);

        if (postContent.trim() === "" && !mediaFile) {
            alert("Write something or upload an image/video!");
            return;
        }

        let post = {
            author: currentUser,
            content: postContent,
            media: mediaFile ? URL.createObjectURL(mediaFile) : null,
            mediaType: mediaFile ? mediaFile.type : null,
            likes: 0,
            comments: [],
            taggedUsers: taggedUsers
        };

        posts.push(post);
        updateFeed();
        updateTrending();
        
        taggedUsers.forEach(user => notifyUser(`${user} was tagged in a post by ${currentUser}`));
        
        document.getElementById("postContent").value = "";
        document.getElementById("mediaUpload").value = "";
        document.getElementById("tagUsers").value = "";
    }

    function updateFeed() {
        let feed = document.getElementById("feed");
        feed.innerHTML = "";
        
        posts.forEach((post, index) => {
            let postContainer = document.createElement("div");
            postContainer.classList.add("post");

            let postText = document.createElement("p");
            postText.innerHTML = `<strong>${post.author}:</strong> ${post.content}`;

            let likeButton = document.createElement("span");
            likeButton.innerHTML = `❤️ Like (${post.likes})`;
            likeButton.classList.add("like-btn");
            likeButton.onclick = function () {
                post.likes++;
                likeButton.innerHTML = `❤️ Like (${post.likes})`;
                notifyUser(`${currentUser} liked a post by ${post.author}`);
                updateTrending();
            };

            let followButton = document.createElement("span");
            followButton.innerHTML = `➕ Follow`;
            followButton.classList.add("follow-btn");
            followButton.onclick = function () {
                if (!users[currentUser].following.includes(post.author)) {
                    users[currentUser].following.push(post.author);
                    users[post.author].followers.push(currentUser);
                    notifyUser(`${currentUser} followed ${post.author}`);
                }
            };

            let mediaElement = post.media ? document.createElement(post.mediaType.startsWith("image/") ? "img" : "video") : null;
            if (mediaElement) {
                mediaElement.src = post.media;
                if (post.mediaType.startsWith("video/")) mediaElement.controls = true;
            }

            postContainer.append(postText, likeButton, followButton, mediaElement);
            feed.prepend(postContainer);
        });
    }

    function updateTrending() {
        let trending = document.getElementById("trending");
        trending.innerHTML = posts.filter(p => p.likes > 0).sort((a, b) => b.likes - a.likes).slice(0, 5).map(p => `<p>🔥 ${p.content} - ${p.likes} likes</p>`).join("");
    }

    function notifyUser(message) {
        let notifications = document.getElementById("notifications");
        let notification = document.createElement("p");
        notification.innerText = message;
        notifications.prepend(notification);
    }
</script>

</body>
</html>
