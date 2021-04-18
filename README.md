# node1
First Node.JS project using simple API

This Node.JS project is based on a simple selection of products where product information is served dynamic via a simple API. You will need to use the following local address to use this application: http://127.0.0.1:8000/     There are currently the following routes specified: /   /overview   /product  /api   as well as an error route and routes for each product according to its ID as specified in the JSON file containing all of the product information. 

This project helped me to learn and better understand:
1) What Node actually is
2) Running JavaScript outside the broswer
3) Using modules
4) Reading and writing files
5) The Blocking and non-blocking nature of Node.JS (Async vs sync)
6) Reading and writing files asynchronously
7) Creating a simple web server
8) Routing
9) Building a simple API
10) HTML Templating - building and filling
11) Parsing variables from URL's

This project is not fully functional as the shopping cart currently has no functionality. The code has been set up and organized to limit the server request to as few as possible. 

A general walk through of the project with notes/observations is as below:

To begin - to read files from the file system we needed to fire import the file system (fs) core module which gives us access to the functions for reading and writing data into the file system.
```JavaScript
const fs = require("fs");
```

Next we needed to import another core module which gives networking capabilities which we need to build our server. 
```JavaScript
const http = require("http");
```

After this we are ready to create our server which accepts a callback function that is fired off on each request - with each new request the callback function is called.
```JavaScript
const server = http.createServer((req, res) => {
  res.end("Hello from the server!");
});
```

Now that we have our server up and running we can listen to the request.
```JavaScript
server.listen(8000, "127.0.0.1", () => {
  console.log("Listening to requests on port 8000");
});
```

Now we can begin setting up routes for different URLs - to be able to analyze and parse URL data we need to import another core module, http. We then use this module to create different pathnames.
```JavaScript
  if (pathName === "/" || pathName === "/overview") {
    res.end("🎇 This is the overview!");
  } else if (pathName === "/product") {
    res.end("💍 This is the product page!");
  } else {
    res.writeHead(404, {
      "Content-type": "text/html", //now browser expects html
      "my-own-header": "hello-bob"
    });
    res.end(`<h1>📛 Page not found!</h1>`); 
  }
});
```

Next was built a very simple API using the data.json file which is one big array containing 5 objects with all of the products data. It it this data that will be sent from the API to the user when a request is recieved. To do this a new route was  added to /api. Next is to read the data from the json file then parse json into javascript and send back that result to the user. 
```JavaScript
const server = http.createServer((req, res) => {
  const pathName = req.url;
  if (pathName === "/" || pathName === "/overview") {
    res.end("🏞 This is the overview!");
  } else if (pathName === "/product") {
    res.end("👢 This is the products page!");
  } else if (pathName === "/api") {
    res.writeHead(200, { "Content-type": "application/json" }); 

    res.end(data); //must send a string back so send data not 'API'
  } else {
    res.writeHead(404, {
      "Content-type": "text/html",
      "my-own-header": "hello-bob",
    });
    res.end(`<h1>❌ Page not found!</h1>`);
  }
});
```

To do this we need to read the json file using JASON.parse() which takes the json code that is a string and turn it into a javascript array of objects. 
```JavaScript
const data = fs.readFileSync(`${__dirname}/dev-data/data.json`, "utf-8");
const dataObject = JSON.parse(data);
```

Next is to send back this data as the response. To do this remember to use a writeHead method which informs the browser to expect application/json as seen above.
```JavaScript
res.end(data); 
```

Recall that if the file system method is used inside of the server that it wil require to get read upon every server request. Moving this method outside of the server and making it synchronous instead ensures that the data is loaded at the start of the application and every request made will result in the data being made available without having to read the file again user request made. This keeps the thread clear and increases efficiency by lowering server request. 
```JavaScript
const data = fs.readFileSync(`${__dirname}/dev-data/data.json`, "utf-8");
```

At this point the API is up and working. It sends back the data contained in the JSON file regarding the five products used in the project. It allows the user to request all the data about the application with a single API call. We did this by adding a route to the api and we also read the file synchronously, put it inside of an object (dataObj) and then sent back that object as a response making sure to specify that we are sending back application/json so the browser is prepared.

Next up is building the UI. This was done by creating three templates - 1 for the overview page of the API, one for the product page of the API and one for the product cards which make up the product container of the API. Since it cannot always be known the number of products, creating a single template allows it to be inserted as many times as required. 

All of the pieces which are to be added dynamically must be prepared by using placeholders in the templates where appropriate. Then later the placeholder will get replaced with the real data. 
```HTML
<h2 class="product__name">{%PRODUCTNAME%}</h2>
        <div class="product__details">
          <p><span class="emoji-left">🌍</span>From {%FROM%}</p>
          <p><span class="emoji-left">❤️</span>{%NUTRIENTS%}</p>
          <p><span class="emoji-left">📦</span>{%QUANTITY%}</p>
          <p><span class="emoji-left">🏷</span>{%PRICE%}€</p>
```

This process was repeated for all three templates. Special note should be given to the anchor class inside of template-cards as the href is what will dynamically set the product number so as to retrieve and display the correct data
```HTML
<a class="card__link" href="/product?id={%ID%}">
```

After all of the templates have had their placeholders inserted then the next thing to do is fill them up. To do this we will create variables for each of the templates. 
These are sync functions bc they are top-level.
```JavaScript
const tempOverview = fs.readFileSync(
  `${__dirname}/templates/template-overview.html`,
  'utf-8'
);
const tempCard = fs.readFileSync(
  `${__dirname}/templates/template-card.html`,
  'utf-8'
);
const tempProduct = fs.readFileSync(
  `${__dirname}/templates/template-product.html`,
  'utf-8'
);
```
Now the dataObj array will be looped over and for each of them the placeholders in the template will be replaced with the data from the  product. The placeholders use the //g method to ensure that placeholders with the same name are changed, not only the first occurence.
```JavaScript
   const cardsHtml = dataObj.map(el => replaceTemplate(tempCard, el)).join('');
    const output = tempOverview.replace('{%PRODUCT_CARDS%}', cardsHtml);
    res.end(output);
```

A replaceTemplate function was created to replace the placeholders with the products.
```JavaScript
const replaceTemplate = (temp, product) => {
  let output = temp.replace(/{%PRODUCTNAME%}/g, product.productname);
  output = output.replace(/{%IMAGE%}/g, product.image);
  output = output.replace(/{%PRICE%}/g, product.price);
  output = output.replace(/{%FROM%}/g, product.from);
  output = output.replace(/{%NUTRIENTS%}/g, product.nutrients);
  output = output.replace(/{%QUANTITY%}/g, product.quantity);
  output = output.replace(/{%DESCRIPTION%}/g, product.description);
  output = output.replace(/{%ID%}/g, product.id);

  if (!product.organic)
    output = output.replace(/{%NOT_ORGANIC%}/g, 'not-organic');
  return output;
};
```

So to recap so far - we loop over the dataObj array which holds all the products. On each iteration we will replace the placeholders in the template card with the current product. This is done using cardsHTML
```JavaScript
    const cardsHtml = dataObj.map(el => replaceTemplate(tempCard, el)).join('');
    const output = tempOverview.replace('{%PRODUCT_CARDS%}', cardsHtml);
    res.end(output);
```

Now variables needed to be parsed from the URL to build the product page. This uses the URL module. The query and pathname are used to do this because the object has them for property names. So we can use destructuring with the exact property names to create two variables - query and pathname which will have the values assigned to them from the object. 
```JavaScript
  const { query, pathname } = url.parse(req.url, true);
```

Now can build the page using the template! Sheesh! Determine the product to display. dataObj is an array and then the element will be treieved from the position determined by the element ID (0-5 have been assigned)
```JavaScript
   const product = dataObj[query.id];
```

Now can create the output
```JavaScript
 const output = replaceTemplate(tempProduct, product);
    res.end(output);
```

The output gets sent as a response. The back button on the products page is implemented in the template product. Setting it back to overview takes the userback to the overview screen after they have finished viewing the product.
```HTML
    <a href="/overview" class="product__back">
```

Finally the replaceTemplate function was exported and made into a module which is then able to be imported across files as needed. 
```JavaScript
module.exports = (temp, product) => {
  let output = temp.replace(/{%PRODUCTNAME%}/g, product.productname);
  output = output.replace(/{%IMAGE%}/g, product.image);
  output = output.replace(/{%PRICE%}/g, product.price);
  output = output.replace(/{%FROM%}/g, product.from);
  output = output.replace(/{%NUTRIENTS%}/g, product.nutrients);
  output = output.replace(/{%QUANTITY%}/g, product.quantity);
  output = output.replace(/{%DESCRIPTION%}/g, product.description);
  output = output.replace(/{%ID%}/g, product.id);

  if (!product.organic)
    output = output.replace(/{%NOT_ORGANIC%}/g, 'not-organic');
  return output;
};
```

***End walkthrough
