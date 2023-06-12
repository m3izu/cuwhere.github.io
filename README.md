<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>CUWHere</title>
    <style>
        /* Add your custom CSS styles here */
		 body {
            font-family: Arial, sans-serif;
            background-color: #f2f2f2;
            margin: 0;
            padding: 0;
        }
		    header {
        background-color: #00695C;
        color: #fff;
        padding: 20px;
        text-align: center;
    }

    h1 {
        margin: 0;
    }

    main {
        padding: 20px;
        max-width: 800px;
        margin: 0 auto;
    }

    section {
        margin-bottom: 40px;
    }

    h2 {
        margin-top: 0;
    }

    form {
        display: grid;
        gap: 10px;
    }

    label,
    input,
    button {
        width: 100%;
        padding: 10px;
    }

    label {
        font-weight: bold;
    }

    button {
        background-color: #00695C;
        color: #fff;
        border: none;
        cursor: pointer;
        transition: background-color 0.3s ease-in-out;
    }

    button:hover {
        background-color: #004D40;
    }

    ul {
        list-style: none;
        padding: 0;
        margin: 0;
    }

    li {
        background-color: #fff;
        padding: 20px;
        margin-bottom: 20px;
        border-radius: 5px;
        box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
    }

    h3 {
        margin-top: 0;
    }

    img {
        max-width: 100%;
        height: auto;
        margin-top: 10px;
    }

</style>

    </style>
</head>

<body>
    <header>
        <h1>CUwWhere</h1>
    </header>
    <main>
        <section>
            <h2>Upload a Coffee Item</h2>
            <form id="coffeeForm">
                <label for="coffeeName">Coffee Name</label>
                <input type="text" id="coffeeName" required>
                <label for="brand">Brand</label>
                <input type="text" id="brand" required>
                <label for="roast">Roast</label>
                <input type="text" id="roast" required>
                <label for="image">Image</label>
                <input type="file" id="image" accept="image/*" required>
                <button type="submit">Upload</button>
            </form>
        </section>
        <section>
            <h2>Recent Coffee Items</h2>
            <ul id="coffeeList"></ul>
        </section>
    </main>
    <script src="https://www.gstatic.com/firebasejs/8.6.5/firebase-app.js"></script>
    <script src="https://www.gstatic.com/firebasejs/8.6.5/firebase-database.js"></script>
    <script src="https://www.gstatic.com/firebasejs/8.6.5/firebase-storage.js"></script>
    <script>
        // Replace with your Firebase configuration
        const firebaseConfig = {
  apiKey: "API KEY",
  authDomain: "AUTHDOMAIN",
  projectId: "PROJECTID",
  storageBucket: "STORAGEBUCKET",
  messagingSenderId: "MESSAGESENDEERID",
  appId: "APP ID",
        };

        // Initialize Firebase
        firebase.initializeApp(firebaseConfig);

        // Get a reference to the Firebase database
        const database = firebase.database();
        const storage = firebase.storage();

        // Get references to the HTML elements
        const coffeeForm = document.getElementById('coffeeForm');
        const coffeeList = document.getElementById('coffeeList');

        // Upload form submission
        coffeeForm.addEventListener('submit', (e) => {
            e.preventDefault();

            // Get form values
            const coffeeName = document.getElementById('coffeeName').value;
            const brand = document.getElementById('brand').value;
            const roast = document.getElementById('roast').value;
            const image = document.getElementById('image').files[0];

            // Upload image to Firebase storage
            const storageRef = storage.ref(`coffeeImages/${image.name}`);
            const uploadTask = storageRef.put(image);

            uploadTask.on('state_changed', (snapshot) => {
                // Handle upload progress
                const progress = (snapshot.bytesTransferred / snapshot.totalBytes) * 100;
                console.log(`Upload progress: ${progress}%`);
            }, (error) => {
                console.error(error);
            }, () => {
                // Image upload complete
                uploadTask.snapshot.ref.getDownloadURL().then((downloadURL) => {
                    // Save coffee data to database
                    const coffeeData = {
                        coffeeName: coffeeName,
                        brand: brand,
                        roast: roast,
                        image: downloadURL
                    };
                    const newCoffeeRef = database.ref('coffeeItems').push();
                    newCoffeeRef.set(coffeeData).then(() => {
                        console.log('Coffee item uploaded successfully');
                        // Reset form
                        coffeeForm.reset();
                    }).catch((error) => {
                        console.error(error);
                    });
                }).catch((error) => {
                    console.error(error);
                });
            });
        });

        // Load recent coffee items from database
        const coffeeItemsRef = database.ref('coffeeItems');
        coffeeItemsRef.on('child_added', (snapshot) => {
            const coffeeData = snapshot.val();
            const coffeeItem = `
                <li>
                    <h3>${coffeeData.coffeeName}</h3>
                    <p><strong>Brand:</strong> ${coffeeData.brand}</p>
                    <p><strong>Roast:</strong> ${coffeeData.roast}</p>
                    <img src="${coffeeData.image}" alt="${coffeeData.coffeeName}">
                </li>
            `;
            coffeeList.insertAdjacentHTML('beforeend', coffeeItem);
        });

    </script>
</body>

</html>

