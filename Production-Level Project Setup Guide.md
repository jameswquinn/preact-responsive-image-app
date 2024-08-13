# Preact App with Responsive Images - Production-Level Setup Guide

This guide provides a production-ready setup for a Preact application with advanced responsive image handling.

## Enhanced Project Structure

```
project-root/
├── src/
│   ├── components/
│   │   ├── ResponsiveImage/
│   │   │   ├── ResponsiveImage.js
│   │   │   └── ResponsiveImage.test.js
│   │   └── App/
│   │       ├── App.js
│   │       └── App.test.js
│   ├── utils/
│   │   └── imageUtils.js
│   ├── styles/
│   │   └── global.css
│   ├── index.js
│   └── index.html
├── public/
│   └── images/
│       └── example.png
├── config/
│   ├── webpack.common.js
│   ├── webpack.dev.js
│   └── webpack.prod.js
├── scripts/
│   └── optimize-images.js
├── package.json
├── vercel.json
├── .gitignore
├── .env.example
├── .eslintrc.js
├── .prettierrc
├── jest.config.js
└── README.md
```

## Enhanced File Contents

### 1. package.json

```json
{
  "name": "preact-responsive-image-app",
  "version": "1.0.0",
  "description": "A production-ready Preact app with responsive image handling",
  "main": "src/index.js",
  "scripts": {
    "start": "webpack serve --config config/webpack.dev.js",
    "build": "webpack --config config/webpack.prod.js",
    "test": "jest",
    "lint": "eslint src/**/*.js",
    "format": "prettier --write 'src/**/*.{js,css}'",
    "optimize-images": "node scripts/optimize-images.js",
    "deploy": "vercel"
  },
  "dependencies": {
    "preact": "^10.11.3",
    "preact-router": "^4.1.0"
  },
  "devDependencies": {
    "@babel/core": "^7.20.12",
    "@babel/preset-env": "^7.20.2",
    "@babel/plugin-transform-react-jsx": "^7.20.13",
    "@testing-library/preact": "^3.2.3",
    "babel-loader": "^9.1.2",
    "clean-webpack-plugin": "^4.0.0",
    "css-loader": "^6.7.3",
    "css-minimizer-webpack-plugin": "^4.2.2",
    "dotenv-webpack": "^8.0.1",
    "eslint": "^8.33.0",
    "eslint-plugin-preact": "^0.1.2",
    "file-loader": "^6.2.0",
    "html-webpack-plugin": "^5.5.0",
    "jest": "^29.4.1",
    "mini-css-extract-plugin": "^2.7.2",
    "prettier": "^2.8.3",
    "responsive-loader": "^3.1.2",
    "sharp": "^0.31.3",
    "style-loader": "^3.3.1",
    "terser-webpack-plugin": "^5.3.6",
    "webpack": "^5.75.0",
    "webpack-cli": "^5.0.1",
    "webpack-dev-server": "^4.11.1",
    "webpack-merge": "^5.8.0"
  }
}
```

### 2. config/webpack.common.js

```javascript
const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');
const { CleanWebpackPlugin } = require('clean-webpack-plugin');
const sharp = require('sharp');
const fs = require('fs').promises;

module.exports = {
  entry: './src/index.js',
  output: {
    path: path.resolve(__dirname, '../dist'),
    filename: '[name].[contenthash].js',
    publicPath: '/',
  },
  module: {
    rules: [
      {
        test: /\.js$/,
        exclude: /node_modules/,
        use: {
          loader: 'babel-loader',
          options: {
            presets: ['@babel/preset-env'],
            plugins: [
              ['@babel/plugin-transform-react-jsx', { pragma: 'h' }]
            ]
          }
        }
      },
      {
        test: /\.css$/,
        use: ['style-loader', 'css-loader'],
      },
      {
        test: /\.png$/,
        use: [
          {
            loader: 'responsive-loader',
            options: {
              adapter: async (imagePath) => {
                const sharpImage = sharp(imagePath);
                const metadata = await sharpImage.metadata();
                const hasAlpha = metadata.channels === 4;

                const sizes = [300, 600, 1200, 2000];
                const formats = ['webp'];
                if (hasAlpha) {
                  formats.push('png');
                } else {
                  formats.push('jpg');
                }

                for (const format of formats) {
                  for (const size of sizes) {
                    const resizedImage = sharpImage.clone().resize(size);
                    const outputPath = path.join(
                      __dirname,
                      '../dist',
                      'assets',
                      'images',
                      format,
                      `${path.basename(imagePath, '.png')}-${size}.${format}`
                    );
                    
                    if (format === 'webp') {
                      await resizedImage.webp({ quality: 80 }).toFile(outputPath);
                    } else if (format === 'png') {
                      await resizedImage.png({ quality: 100 }).toFile(outputPath);
                    } else {
                      await resizedImage.jpeg({ quality: 85 }).toFile(outputPath);
                    }
                  }
                }

                await fs.writeFile(
                  path.join(__dirname, '../dist', 'assets', 'images', 'metadata', `${path.basename(imagePath, '.png')}.json`),
                  JSON.stringify({ hasAlpha })
                );

                return sharpImage;
              },
              name: '[name]-[width].[ext]',
              outputPath: 'assets/images',
              sizes: [300, 600, 1200, 2000],
              placeholder: true,
              placeholderSize: 20,
            },
          },
        ],
      },
    ],
  },
  plugins: [
    new CleanWebpackPlugin(),
    new HtmlWebpackPlugin({
      template: 'src/index.html',
      minify: {
        collapseWhitespace: true,
        removeComments: true,
        removeRedundantAttributes: true,
        useShortDoctype: true,
      },
    }),
  ],
};
```

### 3. config/webpack.dev.js

```javascript
const { merge } = require('webpack-merge');
const common = require('./webpack.common.js');
const Dotenv = require('dotenv-webpack');

module.exports = merge(common, {
  mode: 'development',
  devtool: 'inline-source-map',
  devServer: {
    historyApiFallback: true,
    open: true,
    compress: true,
    hot: true,
    port: 8080,
  },
  plugins: [
    new Dotenv({
      path: './.env.development',
    }),
  ],
});
```

### 4. config/webpack.prod.js

```javascript
const { merge } = require('webpack-merge');
const common = require('./webpack.common.js');
const MiniCssExtractPlugin = require('mini-css-extract-plugin');
const CssMinimizerPlugin = require('css-minimizer-webpack-plugin');
const TerserPlugin = require('terser-webpack-plugin');
const Dotenv = require('dotenv-webpack');

module.exports = merge(common, {
  mode: 'production',
  devtool: 'source-map',
  module: {
    rules: [
      {
        test: /\.css$/,
        use: [MiniCssExtractPlugin.loader, 'css-loader'],
      },
    ],
  },
  plugins: [
    new MiniCssExtractPlugin({
      filename: '[name].[contenthash].css',
    }),
    new Dotenv({
      path: './.env.production',
    }),
  ],
  optimization: {
    minimizer: [
      new CssMinimizerPlugin(),
      new TerserPlugin({
        parallel: true,
      }),
    ],
    runtimeChunk: 'single',
    splitChunks: {
      cacheGroups: {
        vendor: {
          test: /[\\/]node_modules[\\/]/,
          name: 'vendors',
          chunks: 'all',
        },
      },
    },
  },
  performance: {
    hints: 'warning',
    maxEntrypointSize: 512000,
    maxAssetSize: 512000,
  },
});
```

### 5. src/components/ResponsiveImage/ResponsiveImage.js

```javascript
import { h } from 'preact';
import { useState, useEffect } from 'preact/hooks';
import { loadImageMetadata } from '../../utils/imageUtils';

const ResponsiveImage = ({ src, alt, sizes }) => {
  const [imageMeta, setImageMeta] = useState(null);
  const [error, setError] = useState(null);

  useEffect(() => {
    loadImageMetadata(src)
      .then(setImageMeta)
      .catch(err => {
        console.error('Failed to load image metadata:', err);
        setError('Failed to load image. Please try again later.');
      });
  }, [src]);

  if (error) return <div class="error">{error}</div>;
  if (!imageMeta) return <div class="loading">Loading...</div>;

  const basePath = '/assets/images';
  const webpSrcSet = [300, 600, 1200, 2000]
    .map(size => `${basePath}/webp/${src.replace(/\.[^/.]+$/, "")}-${size}.webp ${size}w`)
    .join(', ');
  
  const fallbackFormat = imageMeta.hasAlpha ? 'png' : 'jpg';
  const fallbackSrcSet = [300, 600, 1200, 2000]
    .map(size => `${basePath}/${fallbackFormat}/${src.replace(/\.[^/.]+$/, "")}-${size}.${fallbackFormat} ${size}w`)
    .join(', ');

  return (
    <img
      src={`${basePath}/${fallbackFormat}/${src.replace(/\.[^/.]+$/, "")}-600.${fallbackFormat}`}
      srcSet={`${webpSrcSet}, ${fallbackSrcSet}`}
      sizes={sizes}
      alt={alt}
      loading="lazy"
    />
  );
};

export default ResponsiveImage;
```

### 6. src/utils/imageUtils.js

```javascript
export const loadImageMetadata = async (src) => {
  try {
    const response = await fetch(`/assets/images/metadata/${src.replace(/\.[^/.]+$/, "")}.json`);
    if (!response.ok) throw new Error('Failed to load image metadata');
    return await response.json();
  } catch (error) {
    console.error('Error loading image metadata:', error);
    throw error;
  }
};
```

### 7. src/components/App/App.js

```javascript
import { h } from 'preact';
import { Router } from 'preact-router';
import ResponsiveImage from '../ResponsiveImage/ResponsiveImage';

const Home = () => (
  <div>
    <h1>Preact Responsive Image Demo</h1>
    <ResponsiveImage
      src="example.png"
      alt="Example responsive image"
      sizes="(max-width: 600px) 300px, (max-width: 1200px) 600px, 1200px"
    />
  </div>
);

const App = () => (
  <div id="app">
    <Router>
      <Home path="/" />
    </Router>
  </div>
);

export default App;
```

### 8. src/index.js

```javascript
import { h, render } from 'preact';
import App from './components/App/App';
import './styles/global.css';

render(<App />, document.getElementById('app'));
```

### 9. .env.example

```
API_URL=https://api.example.com
```

### 10. .eslintrc.js

```javascript
module.exports = {
  env: {
    browser: true,
    es2021: true,
    node: true,
    jest: true,
  },
  extends: [
    'eslint:recommended',
    'plugin:preact/recommended',
  ],
  parserOptions: {
    ecmaVersion: 12,
    sourceType: 'module',
  },
  rules: {
    // Add custom rules here
  },
};
```

### 11. .prettierrc

```json
{
  "singleQuote": true,
  "trailingComma": "es5",
  "tabWidth": 2,
  "semi": true,
  "printWidth": 100
}
```

### 12. jest.config.js

```javascript
module.exports = {
  testEnvironment: 'jsdom',
  transform: {
    '^.+\\.js$': 'babel-jest',
  },
  moduleNameMapper: {
    '\\.(css|less|scss|sass)$': 'identity-obj-proxy',
  },
  setupFilesAfterEnv: ['<rootDir>/src/setupTests.js'],
};
```

### 13. scripts/optimize-images.js

```javascript
const sharp = require('sharp');
const fs = require('fs').promises;
const path = require('path');

const inputDir = path.join(__dirname, '../public/images');
const outputDir = path.join(__dirname, '../src/assets/images');

async function optimizeImages() {
  try {
    const files = await fs.readdir(inputDir);
    
    for (const file of files) {
      if (path.extname(file).toLowerCase() === '.png') {
        const inputPath = path.join(inputDir, file);
        const image = sharp(inputPath);
        const metadata = await image.metadata();
        
        // Generate WebP
        await image
          .webp({ quality: 80 })
          .toFile(path.join(outputDir, 'webp', `${path.basename(file, '.png')}.webp`));
        
        // Generate PNG or JPEG based on alpha channel
        if (metadata.channels === 4) {
          await image
            .png({ quality: 100 })
            .toFile(path.join(outputDir, 'png', file));
        } else {
          await image
            .jpeg({ quality: 85 })
            .toFile(path.join(outputDir, 'jpeg
