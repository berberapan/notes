##### Prerequisites
- Node.js
- Yarn package manager

##### Creating the project
To create the boilerplate you first type this in the terminal

```terminal
yarn create vite
```

This will fetch and configure the tools for the project. Then it will ask you to name the project. 

After that it will ask you to select framework were you should select React. Then it will ask you for which language to use and in this example we're using JavaScript.

CD into the project directory and then type yarn to install the dependencies. 

```terminal
yarn
```

##### Starting the server
From the project folder 

```terminal
yarn run dev
```

It will spin up the local server. Default port for the React project is 5173.
http://localhost:5173

If you want to access it on the phone you have to get the network address. From the project folder

```terminal
yarn run dev --host
```

Make sure the phone is connected to the same connection.

##### Remove boilerplate files

Remove the following files

- src/App.css
- src/App.jsx
- src/index.css
- src/assets (whole directory)

And remove the `import "./index.cc"` from the src/main.jsx file.


[Set up project](https://www.digitalocean.com/community/tutorials/how-to-set-up-a-react-project-with-vite)

