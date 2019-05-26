# Study-BuilderBook
Learn React,Nextjs,Express,Material-UI,Jss,and so on...

#### Chapter 1
* Server-side render(SSR)
> Next.js renders pages on the server(for initial load);however,server-side rendering does not come out-of-the-box in Material-UI.After we successfully inject styles to our HtML on the server,we'll need to remove the server-side styles once the client injects styles(client-side styles)

> To understand the SSR,you should know:
- [x] Reactjs https://reactjs.org/docs/hello-world.html
- [x] Nextjs https://nextjs.org/docs
- [x] css in js (React-Jss) https://github.com/cssinjs/react-jss
- [x] Material-UI https://material-ui.com/getting-started/usage/

> index.js include a `button` from Material-UI

``` javascript
// pages/index.js
import Button from '@material-ui/core/Button';
import withLayout from '../lib/withLayout';
const Index = () => (
  <div style={{ padding: "10px 45px" }}>
    <Head>
      <title>Index page</title>
      <meta
        name="description"
        content="This is the description fo the Index page"
      />
    </Head>
    <p>content on Index page</p>
    <Button variant="contained">
      MUI button
    </Button>
  </div>
);
export default withLayout(Index)
```

> context.js include the `theme` and the `pageContext` methods 

```javascript
// lib/context.js
import { SheetsRegistry } from "react-jss";
import {
  createMuiTheme,
  createGenerateClassName
} from "@material-ui/core/styles";
import blue from "@material-ui/core/colors/blue";
import grey from "@material-ui/core/colors/grey";
const theme = createMuiTheme({
  palette: {
    primary: { main: blue[700] },
    secondary: { main: grey[700] }
  },
  typography: {
    useNextVariants: true
  }
});
function createPageContext() {
  return {
    theme,
    sheetsManager: new Map(),
    sheetsRegistry: new SheetsRegistry(),
    generateClassName: createGenerateClassName()
  };
}

export default function getContext() {
  if (!process.browser) {
    return createPageContext();
  }
  if (!global.INIT_MATERIAL_UI) {
    global.INIT_MATERIAL_UI = createPageContext();
  }
  console.log(global.INIT_MATERIAL_UI);
  return global.INIT_MATERIAL_UI;
}
```
> By customizing class `_document` and function `renderPage`,render the `button` on server side 

```javascript
//pages/_document.js
import Document, { Head, Main, NextScript } from "next/document";
import React from "react";
import JssProvider from "react-jss/lib/JSSProvider";
import getContext from "../lib/context";

class MyDocument extends Document {
  render() {
    return (
      <html lang="en">
        <Head>
          <meta charSet="utf-8" />
          <meta
            name="viewport"
            content="width=device-width,initial-scale=1.0"
          />
          <meta name="google" content="notranslate" />
          <meta name="theme-color" content="#1976D2" />
          <link
            rel="shortcut icon"
            href="https://storage.googleapis.com/builderbook/favicon32.png"
          />
          <link
            rel="stylesheet"
            href="https://fonts.googleapis.com/css?family=Muli:300,400:latin"
          />
          <link
            rel="stylesheet"
            href="https://fonts.googleapiscom/icon?family=material+Icons"
          />
          <link
            rel="stylesheet"
            href="https://storage.googleapis.com/builderbook/nprogress.min.css"
          />
          <link
            rel="stylesheet"
            href="https://storage.googleapis.com/builderbook/vs.min.css"
          />
          <style>
            {`
                            a,a:focus{
                                font-weight:400
                                color:#1565c0;
                                text-decoration:none
                                outline:none
                            }
                            a:hover,button:hover{
                                opacity:0.75
                                cursor:ponter
                            }
                            blockquote{
                                padding:0 1em
                                color:#555；
                                border-left:0.25em solid #dfe2e5;
                            }
                            pre{
                                display:block
                                overflow-x:auto
                                padding:0.5em
                                background:#FFF
                                border:1px solid #ddd
                            }
                            code{
                                font-size:14px
                                background:#FFF
                                padding:3px 5px
                            }
                        `}
          </style>
        </Head>
        <body
          style={{
            font: "16px Muli",
            color: "#222",
            margin: "0px auto",
            fontWeight: "300",
            lineHeight: "1.5em",
            bakcgroundColor: "#F7F9FC"
          }}
        >
          <Main />
          <NextScript />
        </body>
      </html>
    );
  }
}

MyDocument.getInitialProps = ({ renderPage }) => {
  const pageContext = getContext();
  const page = renderPage(Component => props => (
    <JssProvider
      registry={pageContext.sheetsRegistry}
      generateClassName={pageContext.generateClassName}
    >
      <Component pageContext={pageContext} {...props} />
    </JssProvider>
  ));
  return {
    ...page,
    pageContext,
    styles: (
      <style
        id="jss-server-side"
        dangerouslySetInnerHTML={{
          __html: pageContext.sheetsRegistry.toString()
        }}
      />
    )
  };
};
export default MyDocument;
```
> After inject styles on server,and after the component is mounted on the browser ,remove the server-side styles

```javascript
//lib/withLayout.js
import React from "react";
import PropTypes from 'prop-types'
import {MuiThemeProvider} from '@material-ui/core/styles';
import getContext from './context';
import CssBaseline from "@material-ui/core/CssBaseline";
import Header from "../components/Header";
function withLayout(BaseComponent) {
  class App extends React.Component {
    constructor(props){
      super(props);
      const {pageContext}=this.props;
      this.pageContext=pageContext || getContext();
    }
    componentDidMount(){
      const jssStyles=document.querySelector('#jss-server-side');
      if(jssStyles&&jssStyles.parentNode){
        jssStyles.parentNode.removeChild(jssStyles);
      }
    }

    render() {
      return (
        <MuiThemeProvider theme={this.pageContext.theme} sheetManager={this.pageContext.sheetsManager}>
          <CssBaseline />
          <div>
             <Header {...this.props} />
             <BaseComponent {...this.props} />
          </div>
        </MuiThemeProvider>
      );
    }
  }
  App.propTypes={
    pageContext:PropTypes.object,
  };
  App.defaultProps={
    pageContext:null,
  };

  return App;
}
export default withLayout;

```
