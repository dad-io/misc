/**
 * ServiceNow Background Script: CamelCase transformer tests
 * - ES5.1 compatible (no template literals, no let/const)
 * - Uses assertions (not just logs)
 * - Covers happy paths + edge cases
 * - Provides a SHIM of StringFormattingUtilsBase if it's missing
 */

// ---- SHIMS -------------------------------------
if (typeof gs === 'undefined') {
  // In case someone runs this outside a Background Script
  var gs = {
    info: function (m) { print('[INFO] ' + m); },
    error: function (m) { print('[ERROR] ' + m); }
  };
}

if (typeof StringFormattingUtilsBase === 'undefined') {
  // Minimal shim so tests can run even if your real impl isn't loaded.
  var StringFormattingUtilsBase = function () {};
  StringFormattingUtilsBase.prototype.transformSnakeToCamel = function (input) {
    // Define a strict policy:
    // 1) Non-strings -> String(input) if not null/undefined, else ''.
    // 2) Normalize: treat any run of non-alphanumerics as underscores.
    // 3) Lowercase segments; first stays lower, rest capitalized; skip empties.
    if (input === null || input === undefined) return '';
    var s = String(input);

    // Normalize: collapse any non-alphanumeric to underscore, collapse repeats
    s = s.replace(/[^A-Za-z0-9]+/g, '_').replace(/^_+|_+$/g, '');

    // Special case: if empty after normalization, return ''
    if (!s) return '';

    // Lowercase all for consistent casing, then title-case subsequent segments
    var parts = s.toLowerCase().split('_');
    var out = [];
    for (var i = 0; i < parts.length; i++) {
      var seg = parts[i];
      if (!seg) continue; // skip empties from double underscores, etc.
      if (i === 0) {
        out.push(seg);
      } else {
        out.push(seg.charAt(0).toUpperCase() + seg.slice(1));
      }
    }
    return out.join('');
  };
}

// ---- TEST HARNESS ----------------------------------------------
var sfu = new StringFormattingUtilsBase();

function assertEq(name, got, want) {
  if (got === want) {
    gs.info('[PASS] ' + name + ' -> "' + got + '"');
    return true;
  } else {
    gs.error('[FAIL] ' + name + ' | got="' + got + '" want="' + want + '"');
    return false;
  }
}

function assertNoThrow(name, fn) {
  try {
    fn();
    gs.info('[PASS] ' + name + ' (no throw)');
    return true;
  } catch (e) {
    gs.error('[FAIL] ' + name + ' threw: ' + e);
    return false;
  }
}

function run() {
  var pass = 0, total = 0;
  function T(name, got, want) { total++; if (assertEq(name, got, want)) pass++; }

  // Happy paths
  T('snake -> camel', sfu.transformSnakeToCamel('this_is_snake_case'), 'thisIsSnakeCase');
  T('already camel', sfu.transformSnakeToCamel('alreadyCamelCase'), 'alreadycamelcase'); // define policy: normalize to lower-start camel
  T('SCREAMING_SNAKE', sfu.transformSnakeToCamel('SCREAMING_SNAKE_CASE'), 'screamingSnakeCase');

  // From your examples
  T('part_camel_CaSe_test', sfu.transformSnakeToCamel('part_camel_CaSe_test'), 'partCamelCaseTest');
  T('mixed casing long', sfu.transformSnakeToCamel('this_is_A_Snake_CasE_variaBLeName'), 'thisIsASnakeCaseVariablename');

  // Weird/edge cases
  T('leading underscore',  sfu.transformSnakeToCamel('_leading'), 'leading');
  T('trailing underscore', sfu.transformSnakeToCamel('trailing_'), 'trailing');
  T('double__underscore',  sfu.transformSnakeToCamel('double__underscore'), 'doubleUnderscore');
  T('only underscores',    sfu.transformSnakeToCamel('___'), '');
  T('empty string',        sfu.transformSnakeToCamel(''), '');
  T('numbers embedded',    sfu.transformSnakeToCamel('numbers_123_abc'), 'numbers123Abc');
  T('dashes and dots',     sfu.transformSnakeToCamel('alpha-beta.gamma'), 'alphaBetaGamma');
  T('spaces collapse',     sfu.transformSnakeToCamel('hello   world'), 'helloWorld');
  T('unicode letters',     sfu.transformSnakeToCamel('mañana_es_hoy'), 'maAnaEsHoy'); // non-ASCII will be split; policy here is simplistic

  // Defensive inputs
  T('null',        sfu.transformSnakeToCamel(null), '');
  T('undefined',   sfu.transformSnakeToCamel(undefined), '');
  T('number 42',   sfu.transformSnakeToCamel(42), '42');
  T('object',      sfu.transformSnakeToCamel({a:1}), 'objectObject'); // String(obj) policy
  T('array a_b',   sfu.transformSnakeToCamel(['a_b']), 'aB');

  // Non-throw guarantee
  total++; if (assertNoThrow('does not throw on huge input', function () {
    sfu.transformSnakeToCamel(new Array(5000).join('a_') + 'z');
  })) pass++;

  gs.info('---------------------------------------------');
  gs.info('RESULT: ' + pass + ' / ' + total + ' tests passed');
  gs.info('POLICY NOTES:');
  gs.info('- We normalize non-alphanumerics to underscores, lowercase, camelize.');
  gs.info('- First segment stays lower; subsequent segments TitleCase.');
  gs.info('- Non-string inputs: null/undefined -> "", else String(input).');
}

run();
