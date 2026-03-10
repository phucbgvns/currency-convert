### Demo non exponent (for vnd)
![Demo GIF](currency-convert.gif)

### Demo exponent (for usd)
![Demo GIF](convert2.gif)

### Note:
Contact me to have full lib for use.

### add lib to file .toml
```toml
[versions]
currencyConvert = "26.11.02"
[libraries]
currncyconvert = { module = "io.github.phucbgvns:currency-convert", version.ref = "currencyConvert" }
```
### add lib to file .gradle
```gradle
kotlin {
    sourceSets {
        val commonMain by getting {
            dependencies {
                implementation(libs.currncyconvert)
            }
        }
    }
}
````
### Integrate to layout
```kotlin
import com.phucdx.currency.api.calc.CurrencyCalcFact
import com.phucdx.currency.api.calc.ICurrencyCalc
import com.phucdx.currency.api.toword.ICurrencyToWord
import com.phucdx.currency.api.format.CurrencyFormatFact
import com.phucdx.currency.api.format.ICurrencyFormater
import com.phucdx.currency.api.toword.CurrencyToWordFact
import androidx.compose.foundation.interaction.MutableInteractionSource
import androidx.compose.foundation.text.KeyboardActions
import androidx.compose.foundation.text.KeyboardOptions
import androidx.compose.material3.LocalTextStyle
import androidx.compose.material3.OutlinedTextField
import androidx.compose.material3.OutlinedTextFieldDefaults
import androidx.compose.material3.Text
import androidx.compose.material3.TextFieldColors
import androidx.compose.runtime.Composable
import androidx.compose.runtime.collectAsState
import androidx.compose.ui.Modifier
import androidx.compose.ui.graphics.Color
import androidx.compose.ui.graphics.Shape
import androidx.compose.ui.text.AnnotatedString
import androidx.compose.ui.text.TextLayoutResult
import androidx.compose.ui.text.TextStyle
import androidx.compose.ui.text.font.FontFamily
import androidx.compose.ui.text.font.FontStyle
import androidx.compose.ui.text.font.FontWeight
import androidx.compose.ui.text.input.VisualTransformation
import androidx.compose.ui.text.style.TextAlign
import androidx.compose.ui.text.style.TextDecoration
import androidx.compose.ui.text.style.TextOverflow
import androidx.compose.ui.unit.TextUnit
import androidx.compose.ui.unit.sp
import kotlinx.coroutines.flow.MutableStateFlow
import androidx.compose.foundation.layout.Column
import androidx.compose.foundation.layout.fillMaxWidth
import androidx.compose.foundation.layout.padding
import androidx.compose.material3.LocalTextStyle
import androidx.compose.material3.TextField
import androidx.compose.material3.TextFieldDefaults
import androidx.compose.ui.input.key.KeyEvent
import androidx.compose.ui.input.key.onKeyEvent
import androidx.compose.ui.platform.LocalSoftwareKeyboardController
import androidx.compose.ui.text.input.ImeAction
import androidx.compose.ui.text.input.KeyboardType
import androidx.compose.ui.text.input.TextFieldValue
import androidx.compose.ui.unit.dp
import androidx.compose.ui.unit.sp
import org.jetbrains.compose.resources.StringResource
import org.jetbrains.compose.resources.stringResource
import com.phucdx.currency.core.ext.format.KeyPress
import com.phucdx.currency.core.ext.format.TextData

@Composable
fun HomeScreen(
    nav: NavController,
    vm: HomeVm
) {
    MyScaffold(nav) {
        AmountInputWrap(vm)
    }
}

@Immutable
data class UiState(
    val textFieldValue: TextFieldValue = TextFieldValue(
        text = "0",
        selection = TextRange(0)
    ),
)
class HomeVm : ViewModel(), KoinComponent {
    val _uiState = MutableStateFlow(UiState(selectedDay = lunarToday))
    val uiState = _uiState.asStateFlow()
    val numberFormater = CurrencyFormatFact().getCurrencyFormat()
    val currencyToWord = CurrencyToWordFact().getCurrencyToWord()
    val currencyCalc = CurrencyCalcFact().getCurrencyCalc()
    
    fun setTextFieldValue(textFieldValue: TextFieldValue) {
        _uiState.update {
            it.copy(
                textFieldValue = textFieldValue
            )
        }
    }
    
    fun currencyToWord(): String {
        return currencyToWord.currencyToWords(
            uiState.value.textFieldValue.text,
            CurrencyData.LOCALE_VN,
            CurrencyData.VND.codeString
        )
    }
    
    fun onKeyEventBalance(keyCode: KeyEvent): Boolean {
        if (keyCode.key == Key.Backspace) {
            numberFormater.onKey(KeyPress.Backspace)?.also { textData ->
                setTextFieldValue(
                    TextFieldValue(
                        text = textData.text,
                        selection = TextRange(textData.selection, textData.selection)
                    )
                )
            }
            return true
        }
        return false
    }

    fun setAmount(textUpdating: TextData) {
        numberFormater.convert(textUpdating = textUpdating)
            ?.also { textData ->
                _uiState.update {
                    it.copy(
                        textFieldValue = TextFieldValue(
                            text = textData.text,
                            selection = TextRange(textData.selection, textData.selection)
                        )
                    )
                }
            }
    }
}

@Composable
fun AmountInputWrap(vm: HomeVm) {
    val textField = vm.uiState
        .map { it.textFieldValue }
        .collectAsState(vm.uiState.value.textFieldValue).value
    AmountInput(
        textField = textField,
        placeholder = Res.string.confirm_exit,
        label = Res.string.notice,
        currencyToWord = vm.currencyToWord(),
        amountText = vm.uiState.value.textFieldValue.text,
        onKeyEvent = vm::onKeyEventBalance
    ) { textDataChange -> vm.setAmount(textDataChange) }
}

@Composable
fun AmountInput(
    textField: TextFieldValue,
    placeholder: StringResource,
    label: StringResource,
    currencyToWord: String,
    amountText: String,
    onKeyEvent: (KeyEvent) -> Boolean,
    onValueChange: (textDataChange: TextData) -> Unit
) {
    val keyboardController = LocalSoftwareKeyboardController.current
    Column {
        TextField(
            colors = TextFieldDefaults.colors(
                focusedContainerColor = Color.Transparent,
                unfocusedContainerColor = Color.Transparent,
                disabledContainerColor = Color.Transparent,
            ),
            value = textField,
            onValueChange = { value: TextFieldValue ->
                val textDataChange = TextData(
                    text = value.text.replace("-", ""),
                    selection = value.selection.start
                )
                onValueChange(textDataChange)
            },
            placeholder = {
                MyText(stringResource(placeholder))
            },
            label = { MyText(stringResource(label)) },
            singleLine = true,
            maxLines = 1,
            keyboardActions = KeyboardActions(onDone = {
//                focusManager.clearFocus()
                keyboardController?.hide()
            }),
            keyboardOptions = KeyboardOptions(keyboardType = KeyboardType.Number).copy(imeAction = ImeAction.Done),
            modifier = Modifier
                .fillMaxWidth()
                .padding(top = 12.dp)
                .onKeyEvent {
                    onKeyEvent(it)
                },
            textStyle = LocalTextStyle.current.copy(fontSize = 15.sp),
        )
        if (amountText != "0") {
            MyText(currencyToWord)
        }
    }
}

@Composable
fun MyText(
    text: String,
    modifier: Modifier = Modifier,
    color: Color = Color.Unspecified,
    fontSize: TextUnit = 16.sp,
    fontStyle: FontStyle? = null,
    fontWeight: FontWeight? = null,
    fontFamily: FontFamily? = FontFamily.SansSerif,
    letterSpacing: TextUnit = TextUnit.Unspecified,
    textDecoration: TextDecoration? = null,
    textAlign: TextAlign? = null,
    lineHeight: TextUnit = 22.sp,
    overflow: TextOverflow = TextOverflow.Clip,
    softWrap: Boolean = true,
    maxLines: Int = Int.MAX_VALUE,
    minLines: Int = 1,
    onTextLayout: ((TextLayoutResult) -> Unit)? = null,
    style: TextStyle = LocalTextStyle.current
) {
    Text(
        text = text,
        modifier = modifier,
        color = color,
        fontSize = fontSize,
        fontStyle = fontStyle,
        fontWeight = fontWeight,
        fontFamily = fontFamily,
        letterSpacing = letterSpacing,
        textDecoration = textDecoration,
        textAlign = textAlign,
        lineHeight = lineHeight,
        overflow = overflow,
        softWrap = softWrap,
        maxLines = maxLines,
        minLines = minLines,
        onTextLayout = onTextLayout,
        style = style,
    )
}
```
### Additional:
```Kotlin
import com.phucdx.currency.api.calc.CurrencyCalcFact
import kotlin.test.Test
import kotlin.test.assertEquals

class MultiDivTest {
    val currencyCalc = CurrencyCalcFact().getCurrencyCalc()
    @Test
    fun example() {
        var data = currencyCalc.divide("1000", "1", 5)

        assertEquals("1000", data)
        data = currencyCalc.divide("12.3", "4.56", 5) // 2.69736

        assertEquals("2.6973", data)
        data = currencyCalc.divide("-10", "4", 3)     // -2.5

        assertEquals("-2.5", data)
        data = currencyCalc.divide("1", "3", 10)      // 0.3333333333

        assertEquals("0.3333333333", data)

        data = currencyCalc.multiply("135", "100") // 13500

        assertEquals("13500", data)
        data = currencyCalc.multiply("12.3", "4.56")  // 56.088

        assertEquals("56.088", data)
        data =currencyCalc.multiply("-1.2", "3")     // -3.6

        assertEquals("-3.6", data)
        data = currencyCalc.divide("-12.3", "4.56", 5) // 2.6973

        assertEquals("-2.6973", data)
        data = currencyCalc.divide("-10", "-4", 3)     // 2.5

        assertEquals("2.5", data)
    }

    @Test
    fun example() {
        checkCalc()
    }

    private fun checkCalc() {
        val tests = listOf(
            // ===== add =====
            "0" to Triple("11", "-11", "add"),
            "10" to Triple("12", "-2", "add"),
            "-10" to Triple("-11", "1", "add"),
            "-10" to Triple("1", "-11", "add"),
            "-4.75" to Triple("-1.5", "-3.25", "add"),
            "1.75" to Triple("-1.5", "3.25", "add"),
            "1.75" to Triple("3.25", "-1.5", "add"),
            "12.75" to Triple("10.5", "2.25", "add"),
            "2.00" to Triple("1.05", "0.95", "add"),
            "0.001" to Triple("0.01", "-0.009", "add"),

            // ===== subtract =====
            "22" to Triple("11", "-11", "sub"),
            "-12" to Triple("-11", "1", "sub"),
            "12" to Triple("11", "-1", "sub"),
            "10" to Triple("-1", "-11", "sub"),
            "12" to Triple("1", "-11", "sub"),
            "1.75" to Triple("-1.5", "-3.25", "sub"),
            "-4.75" to Triple("-1.5", "3.25", "sub"),
            "4.75" to Triple("3.25", "-1.5", "sub"),
            "8.25" to Triple("10.5", "2.25", "sub"),
            "0.10" to Triple("1.05", "0.95", "sub"),
            "0.019" to Triple("0.01", "-0.009", "sub"),
            "0.00001" to Triple("1.0001", "1.00009", "sub")
        )

        println("=== TEST addDecimalStrings & subtractDecimalStrings ===")
        for ((a, c) in tests) {
            val result = when (c.third) {
                "add" -> currencyCalc.add(c.first, c.second)
                else -> currencyCalc.subtract(c.first, c.second)
            }
            println("${c.first} ${if (c.third == "add") "+" else "-"} ${c.second} = $result")
            assertEquals(a, result)
        }
    }
}
```

