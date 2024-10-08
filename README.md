# Preact Responsive Image Project

This project demonstrates a production-ready Preact application with advanced responsive image handling. It includes optimizations, best practices, and features suitable for a production environment.

[![Deploy with Vercel](https://vercel.com/button)](https://vercel.com/new/clone?repository-url=https://github.com/jameswquinn/preact-responsive-image-app)

## Project Overview

The Preact Responsive Image Project showcases:
- Responsive image loading with WebP support and fallbacks
- Intelligent handling of images with alpha channels
- Webpack configuration for development and production
- Code splitting and performance optimizations
- Environment-specific configurations
- Testing setup with Jest
- Linting and formatting with ESLint and Prettier
- Continuous Integration/Continuous Deployment (CI/CD) setup
- Error boundaries for robust error handling

## Project Structure

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

## Setup Instructions

1. Clone the repository:
   ```
   git clone https://github.com/jameswquinn/preact-responsive-image-app.git
   cd preact-responsive-image-app
   ```

2. Install dependencies:
   ```
   npm install
   ```

3. Set up environment variables (see Environment Variables section below).

4. Place your source PNG images in the `public/images/` directory.

5. Start the development server:
   ```
   npm start
   ```

6. For production build:
   ```
   npm run build
   ```

## Environment Variables

This project uses environment variables for configuration. An `.env.example` file is provided in the root directory as a template.

### Setting Up Environment Variables

1. Copy the `.env.example` file and rename it to `.env.development` for your development environment:
   ```
   cp .env.example .env.development
   ```

2. Create another copy for your production environment:
   ```
   cp .env.example .env.production
   ```

3. Edit both `.env.development` and `.env.production` files and replace the placeholder values with your actual configuration values.

4. Never commit these `.env` files to version control. They are already included in `.gitignore`.

### Available Environment Variables

- `APP_NAME`: Name of the application
- `NODE_ENV`: Current environment (development/production)
- `PORT`: Port number for the server
- `MAX_IMAGE_SIZE`: Maximum allowed image size in bytes
- `ALLOWED_IMAGE_TYPES`: Comma-separated list of allowed image file types
- `VERCEL_API_TOKEN`: API token for Vercel deployments
- `DB_HOST`, `DB_USER`, `DB_PASS`, `DB_NAME`: Database connection details (if applicable)
- `REDIS_URL`: URL for Redis cache (if used)
- `LOG_LEVEL`: Logging level (e.g., info, debug, error)
- `ENABLE_WEBP_CONVERSION`: Feature flag for WebP conversion
- `ENABLE_LAZY_LOADING`: Feature flag for image lazy loading
- `GOOGLE_ANALYTICS_ID`: Google Analytics ID (if used)
- `CDN_URL`: URL for Content Delivery Network (if used)
- `JWT_SECRET`: Secret key for JWT authentication
- `CORS_ORIGIN`: Allowed origin for CORS

Refer to the `.env.example` file for a complete list of available environment variables and their descriptions.

## Usage

To use the ResponsiveImage component in your Preact application:

```jsx
import ResponsiveImage from './components/ResponsiveImage/ResponsiveImage';

const MyComponent = () => (
  <ResponsiveImage
    src="example.png"
    alt="An example image"
    sizes="(max-width: 600px) 300px, (max-width: 1200px) 600px, 1200px"
  />
);
```

## How It Works

1. During the build process:
   - WebP versions are generated for all images.
   - PNG versions are generated only for images with an alpha channel.
   - JPEG versions are generated for images without an alpha channel.
   - Metadata JSON files are created for each image, storing information about the alpha channel.

2. The ResponsiveImage component:
   - Fetches the metadata for the image.
   - Sets up the appropriate `srcSet` for WebP and fallback images.
   - Uses an `img` tag with `srcSet` to provide both WebP and fallback versions, allowing the browser to choose the best format.

3. Error handling:
   - The component includes error handling for both image and metadata loading failures.
   - An ErrorBoundary component in App.js catches any errors that occur during rendering.

## Troubleshooting

If you encounter the error "Failed to load image. Please try again later.", follow these steps:

1. Ensure that you have placed your source PNG images in the `public/images/` directory.

2. Check that the Webpack build process is generating the image assets and metadata correctly:
   - After running `npm run build`, check the `dist/assets/images/` directory for generated image files.
   - Look for metadata JSON files in `dist/assets/images/metadata/`.

3. Verify that the image paths in your code match the actual file names:
   ```jsx
   <ResponsiveImage
     src="your-actual-image-filename.png"
     alt="Description of your image"
     sizes="(max-width: 600px) 300px, (max-width: 1200px) 600px, 1200px"
   />
   ```

4. Check the browser's developer tools (Network tab) to see if the image and metadata files are being requested correctly.

5. If using a development server, ensure it's configured to serve the `dist` directory correctly.

6. For production deployments, verify that all image assets and metadata files are being uploaded to your hosting service.

If the issue persists, you may need to modify the ResponsiveImage component to provide more detailed error information. Update the `src/components/ResponsiveImage/ResponsiveImage.js` file with enhanced error logging and handling.

## Project Workflow

The following flowchart illustrates a simplified version of the Preact Responsive Image Project lifecycle:

```mermaid
graph TD
    A[Development] --> B[Local Testing]
    B --> C[Push to GitHub]
    C --> D[CI/CD Pipeline]
    D --> E[Build Process]
    E --> F[Image Processing]
    F --> G[Deploy to Vercel]
    G --> H[Production Release]
    H --> I[Monitor & Feedback]
    I --> A
```

This simplified diagram shows the main stages of the development and deployment process. The actual workflow includes more detailed steps, such as:
- Linting and formatting checks
- Generating different image formats (WebP, PNG, JPEG) based on alpha channel presence
- Creating metadata JSON files for images
- Running post-deployment tests
- Performance monitoring and optimization

## Scripts

- `npm start`: Start development server
- `npm run build`: Build for production
- `npm test`: Run tests
- `npm run lint`: Lint code
- `npm run format`: Format code
- `npm run optimize-images`: Optimize images in the public directory

## Deployment

### Quick Deploy with Vercel

Click the "Deploy with Vercel" button at the top of this README to deploy the project quickly with Vercel.

### Manual Deployment

This project is set up for deployment on Vercel. To deploy manually:

1. Install Vercel CLI: `npm i -g vercel`
2. Run: `vercel`
3. Follow the prompts to link your project and deploy

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

1. Fork the project
2. Create your feature branch (`git checkout -b feature/AmazingFeature`)
3. Commit your changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to the branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

## License

This project is licensed under the MIT License.

## Acknowledgments

- [Preact](https://preactjs.com/) for the lightweight React alternative
- [Sharp](https://sharp.pixelplumbing.com/) for the high-performance image processing
- [Webpack](https://webpack.js.org/) for bundling and build process
- [Vercel](https://vercel.com/) for easy deployment options
