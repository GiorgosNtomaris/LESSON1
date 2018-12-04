(function () {
  var baseLocation;
  if (typeof document !== 'undefined') {
    baseLocation = new URL('./', document.currentScript.src);
  } else if (typeof location !== 'undefined') {
    // Probably worker -- walking subfolders until we will reach root.
    baseLocation = location;
    while (baseLocation.href.includes('/src/')) {
      baseLocation = new URL('..', baseLocation);
    }
  } else {
    throw new Error('Cannot configure SystemJS');
  }

  var PluginBabelPath = 'node_modules/systemjs-plugin-babel/plugin-babel.js';
  var SystemJSPluginBabelPath =
    'node_modules/systemjs-plugin-babel/systemjs-babel-browser.js';
  var PluginBabelCachePath = 'external/systemjs/plugin-babel-cached.js';

  var isCachingPossible = typeof indexedDB !== 'undefined' &&
                          typeof TextEncoder !== 'undefined' &&
                          typeof crypto !== 'undefined' &&
                          typeof crypto.subtle !== 'undefined';

  // When we create a bundle, webpack is run on the source and it will replace
  // require with __webpack_require__. When we want to use the real require,
  // __non_webpack_require__ has to be used.
  // In this target, we don't create a bundle, so we have to replace the
  // occurences of __non_webpack_require__ ourselves.
  function babelPluginReplaceNonWebPackRequire(babel) {
    return {
      visitor: {
        Identifier(path, state) {
          if (path.node.name === '__non_webpack_require__') {
            path.replaceWith(babel.types.identifier('require'));
          }
        },
      },
    };
  }

  SystemJS.config({
    packages: {
      '': {
        defaultExtension: 'js',
      },
    },
    paths: {
      'pdfjs': new URL('src', baseLocation).href,
      'pdfjs-web': new URL('web', baseLocation).href,
      'pdfjs-test': new URL('test', baseLocation).href,
      'pdfjs-lib': new URL('src/pdf', baseLocation).href,
      'core-js': new URL('node_modules/core-js', baseLocation).href,
    },
    meta: {
      '*': {
        scriptLoad: false,
        esModule: true,
        babelOptions: {
          env: false,
          plugins: [babelPluginReplaceNonWebPackRequire],
        },
      },
    },
    map: {
      'plugin-babel': new URL(PluginBabelPath, baseLocation).href,
      'systemjs-babel-build':
        new URL(SystemJSPluginBabelPath, baseLocation).href,
      'plugin-babel-cached': new URL(PluginBabelCachePath, baseLocation).href,
    },
    transpiler: isCachingPossible ? 'plugin-babel-cached' : 'plugin-babel',
  });
})();
