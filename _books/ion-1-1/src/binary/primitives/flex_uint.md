## `FlexUInt`

A variable-length unsigned integer.

The bytes of a `FlexUInt` are written in
[little-endian byte order](https://en.wikipedia.org/wiki/Endianness). This means that the first bytes will contain
the `FlexUInt`'s least significant bits.

The least significant bits in the `FlexUInt` indicate the number of bytes that were used to encode the integer.
If a `FlexUInt` is `N` bytes long, its `N-1` least significant bits will be `0`; a terminal `1` bit will be
in the next most significant position.

All bits that are more significant than the terminal `1` represent the magnitude of the `FlexUInt`.

#### `FlexUInt` encoding of `14`
```
              ┌──── Lowest bit is 1 (end), indicating
              │     this is the only byte.
0 0 0 1 1 1 0 1
└─────┬─────┘
unsigned int 14
```

#### `FlexUInt` encoding of `729`
```
             ┌──── There's 1 zero in the least significant bits, so this
             │     integer is two bytes wide.
            ┌┴┐
0 1 1 0 0 1 1 0  0 0 0 0 1 0 1 1
└────┬────┘      └──────┬──────┘
lowest 6 bits    highest 8 bits
of the unsigned  of the unsigned
integer          integer
```

#### `FlexUInt` encoding of `21,043`
```
            ┌───── There are 2 zeros in the least significant bits, so this
            │      integer is three bytes wide.
          ┌─┴─┐
1 0 0 1 1 1 0 0  1 0 0 1 0 0 0 1  0 0 0 0 0 0 1 0
└───┬───┘        └──────┬──────┘  └──────┬──────┘
lowest 6 bits    next 8 bits of   highest 8 bits
of the unsigned  the unsigned     of the unsigned
integer          integer          integer
```

### FlexUInt Converter
|                                                 | Value                                                                           |
|-------------------------------------------------|---------------------------------------------------------------------------------|
| <label for="decimalNumber">Decimal</label>      | <input type="text" id="decimalNumber" name="decimalNumber" autocomplete="off"/> |
| <label for="hexFlexUInt">FlexUInt (hex)</label> | <input type="text" id="hexFlexUInt" name="hexFlexUInt" autocomplete="off"/>     |
| <label for="binFlexUInt">FlexUInt (bin)</label> | <input type="text" id="binFlexUInt" name="binFlexUInt" autocomplete="off"/>     |

<script>
const decimalNumber = document.getElementById("decimalNumber");
const hexFlexUInt = document.getElementById("hexFlexUInt");
const binFlexUInt = document.getElementById("binFlexUInt");

// To only allow certain keystrokes
// See https://stackoverflow.com/a/61697867/5563569

// Calculates the number of bytes required to write a FlexUInt.
function flexUIntLength(value) {
  if (value == 0) {
    return 1
  }
  return Math.floor(Math.log2(value) / 7 + 1)
}

// Converts a decimal number to FlexUInt hex string 
function toFlexUIntHex(value) {
  var len = flexUIntLength(value);
  if (!len) {
    return
  }
  var d = (value * 2 + 1) << (len - 1);
  var hex = (d).toString(16);
  hex = hex.length % 2 ? '0' + hex : hex;
  return hex.match(/.{1,2}/g).reverse().join(" ").toUpperCase();
}

function hexFlexUIntToDec(value) {
  var hex = value.replaceAll(" ", "")
  hex = hex.length % 2 ? '0' + hex : hex;
  hex = hex.match(/.{1,2}/g).reverse().join("")
  var dec = parseInt(hex, 16)
  if (!dec) {
    return
  }

  var numBytes = (bin.length + 1) / 3
  var expectedBytes = 1
  while (dec % 2 == 0 && dec > 0) {
    dec = dec / 2
    expectedBytes++
    if (expectedBytes > numBytes) {
      return "undefined"
    }
  }
  return (dec - 1) / 2;
}

// Converts a decimal number to FlexUInt binary string 
function toFlexUIntBin(value) {
  var len = flexUIntLength(value);
  if (!len) {
    return
  }
  var d = (value * 2 + 1) << (len - 1);
  var bin = (d).toString(2);
  var padding = bin.length % 8 ? "0".repeat(8 - bin.length % 8) : "";
  return (padding + bin).match(/.{1,8}/g).reverse().join(" ");
}

function binFlexUIntToDec(value) {
  var bin = value.replaceAll(" ", "")
  
  var padding = bin.length % 8 ? "0".repeat(8 - bin.length % 8) : "";
  bin = (padding + bin).match(/.{1,8}/g).reverse().join("")
  var dec = parseInt(bin, 2)
  
  if (!dec) {
    return
  }

  var numBytes = (bin.length + 1) / 9
  var expectedBytes = 1
  while (dec % 2 == 0 && dec > 0) {
    dec = dec / 2
    expectedBytes++
    if (expectedBytes > numBytes) {
      return "undefined"
    }
  }
  return (dec - 1) / 2;
}

decimalNumber.addEventListener("blur", (event) => {
  if (decimalNumber.value == "") {
    return
  }
  var x = parseInt(decimalNumber.value, 10);
  hexFlexUInt.value = toFlexUIntHex(x);
  binFlexUInt.value = toFlexUIntBin(x);
});

hexFlexUInt.addEventListener("blur", (event) => {
  if (hexFlexUInt.value == "") {
    return
  }
  var h = hexFlexUInt.value.replaceAll(" ", "");
  h = h.length % 2 ? "0" + h : h;
  hexFlexUInt.value = h.match(/.{2}/g).join(" ")

  var x = hexFlexUIntToDec(hexFlexUInt.value);
  decimalNumber.value = x;
  binFlexUInt.value = toFlexUIntBin(x);
});

binFlexUInt.addEventListener("blur", (event) => {
  if (binFlexUInt.value == "") {
    return
  }
  var bin = binFlexUInt.value.replaceAll(" ", "");
  var padding = bin.length % 8 ? "0".repeat(8 - bin.length % 8) : "";
  binFlexUInt.value = (padding + bin).match(/.{1,8}/g).join(" ")

  var x = binFlexUIntToDec(binFlexUInt.value);
  decimalNumber.value = x;
  hexFlexUInt.value = toFlexUIntHex(x);
});

var currentInputValue = '';
binFlexUInt.addEventListener('input', function inputInputEventHandler(e) {
    // use target of event to get value of input
    const target = e.target;
    const DIGITS_SPACE_REGEX = /^[01\s]*$/;
    // test if target.value or value of input box now is valid.
    // if value is valid then update currentInputValue 
    // target.value else its is not value and we will
    // ignore this target.value and replace it with 
    // previously valid value currentInputValue
    DIGITS_SPACE_REGEX.test(target.value) 
        ? ( currentInputValue = target.value ) 
        : ( target.value = currentInputValue );
});
</script>


