<template>
    <div class="calculator">
        <div class="display-container">
            <div class="display" :class="{ 'animate-out': isAnimating }">{{ currentExpression || '0' }}</div>
            <div class="result" :class="{ 'animate-in': isAnimating }">{{ result || '' }}</div>
        </div>
        <div class=" buttons">
            <button @click="clear">C</button>
            <button @click="append( '/' )">÷</button>
            <button @click="append( '*' )">×</button>
            <button @click="del">del</button>
            <button @click="append( '7' )">7</button>
            <button @click="append( '8' )">8</button>
            <button @click="append( '9' )">9</button>
            <button @click="append( '-' )">-</button>
            <button @click="append( '4' )">4</button>
            <button @click="append( '5' )">5</button>
            <button @click="append( '6' )">6</button>
            <button @click="append( '+' )">+</button>
            <button @click="append( '1' )">1</button>
            <button @click="append( '2' )">2</button>
            <button @click="append( '3' )">3</button>
            <button @click="sign">±</button>
            <button @click="append( '%' )">%</button>
            <button @click="append( '0' )">0</button>
            <button @click="dot">.</button>
            <button @click="equal">=</button>
        </div>
    </div>
</template>

<script>
export default {
    data ()
    {
        return {
            result: '',
            currentExpression: '0',
            lastResult: null,
            isAnimating: false
        }
    },
    watch: {
        currentExpression ()
        {
            this.calculateResult();
        }
    },
    methods: {
        isOperator ( char )
        {
            return ['+', '-', '*', '/', '%'].includes( char );
        },
        sanitizeExpression ( expression )
        {
            return expression
                .replace( /%(\d|\.)/g, '/100*$1' )
                .replace( /%([+\-*/]|$)/g, '/100$1' )
                .replace( /[+\-*/]+$/, '' );
        },
        calculateResult ()
        {
            if ( !this.currentExpression )
            {
                this.result = '';
                return;
            }
            const sanitized = this.sanitizeExpression( this.currentExpression );

            const hasValidOperator = /([\d.]|\))([+\-*/])(?=[\d.(])/.test( sanitized );

            if ( hasValidOperator )
            {
                try
                {
                    this.result = new Function( 'return ' + sanitized )().toString();
                    this.lastResult = this.result;
                } catch ( error )
                {
                    this.result = '';
                }
            } else
            {
                this.result = /[+\-*/]$/.test( this.currentExpression )
                    ? this.lastResult || ''
                    : '';
            }
        },
        append ( value )
        {
            const isNumber = /^\d$/.test( value );
            if ( isNumber )
            {
                if ( this.currentExpression === "0" )
                {
                    if ( ["+", "*", "/", "%"].includes( value ) )
                    {
                        this.currentExpression += value;
                    }
                    this.currentExpression = value;
                    return;
                }
                const lastChar = this.currentExpression.slice( -1 );
                if ( this.isOperator( lastChar ) )
                {
                    this.currentExpression += value;
                    return;
                }
            }

            if ( this.isOperator( value ) )
            {
                const lastChar = this.currentExpression.slice( -1 );
                if ( !this.currentExpression )
                {
                    if ( value === "-" ) this.currentExpression = value;
                    return;
                }
                if ( this.isOperator( lastChar ) )
                {
                    // Handle negative sign after other operators
                    if ( value === "-" && ["+", "*", "/", "%"].includes( lastChar ) )
                    {
                        this.currentExpression += value;
                    } else
                    {
                        // Replace previous operator
                        this.currentExpression = this.currentExpression.slice( 0, -1 ) + value;
                    }
                } else
                {
                    this.currentExpression += value;
                }
                return;
            }
            this.currentExpression += value;
        },
        clear ()
        {
            this.currentExpression = '0';
            this.result = '';
            this.lastResult = null;
        },
        sign ()
        {
            if ( this.currentExpression )
            {
                this.lastResult = '';
                this.currentExpression = this.currentExpression.startsWith( '-' )
                    ? this.currentExpression.slice( 1 )
                    : `-${this.currentExpression}`;
            }
        },
        percent ()
        {
            if ( !this.currentExpression ) return;
            const parts = this.currentExpression.split( /([+\-*/])/ );
            const lastNumber = parts.pop() || '';

            if ( !isNaN( lastNumber ) )
            {
                const percentage = parseFloat( lastNumber ) / 100;
                parts.push( percentage.toString() );
                this.currentExpression = parts.join( '' );
            }
        },
        dot ()
        {
            if ( !this.currentExpression.includes( '.' ) )
            {
                this.append( '.' );
            }
        },
        async equal ()
        {
            if ( !this.currentExpression ) return;

            const sanitized = this.sanitizeExpression( this.currentExpression );
                
            const hasValidOperator = /([\d.]|\))([+\-*/])(?=[\d.(])/.test( sanitized );

            if ( hasValidOperator )
            {
                try
                {
                    // Calculate the result
                    const calculation = new Function( 'return ' + sanitized );
                    const finalResult = calculation().toString();

                    // Trigger Animation
                    this.isAnimating = true;
                    await new Promise( ( resolve ) => setTimeout( resolve, 500 ) );

                    // Update the display with the final result
                    this.currentExpression = finalResult;
                    this.result = '';
                    this.lastResult = finalResult;

                } catch ( error )
                {
                    this.result = 'Error';
                } finally
                {
                    this.isAnimating = false;
                }
            }
        },
        del ()
        {
            this.lastResult = '';
            this.currentExpression = this.currentExpression.slice( 0, -1 );
        }
    }
}
</script>

<style scoped>
.display-container {
    position: relative;
    height: 120px;
    overflow: hidden;
    background: #222;
}

.calculator {
    width: 300px;
    margin: 0 auto;
    font-size: 40px;
    display: grid;
}

.display,
.result {
    background: #222;
    color: white;
    padding: 1px 20px;
    text-align: right;
    font-size: 30px;
    transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
}

.result {
    font-size: 20px;
}

.display.animate-out {
    transform: translateY(0%);
    opacity: 0;
    pointer-events: none;
}

.result.animate-in {
    font-size: 30px;
    transform: translateY(-100%);
    opacity: 1;
    pointer-events: auto;
}

.buttons {
    display: grid;
    grid-template-columns: repeat(4, 1fr);
    grid-gap: 15px;
    padding: 15px;
    background: #999;
}

button {
    padding: 20px;
    font-size: 30px;
    border-radius: 5px;
    border: none;
    cursor: pointer;
}

.zero {
    grid-column: span 2;
}
</style>