<template>
    <div id="app">
        <div class="formItem--password">
            <label class="formItem__title" for="newPassword">Jelszó</label>
            <div class="formItem__note">{{noteText}}</div>
            <div class="formItem__password">
                <div class="passwordStrength" :class="'str_' + strengthLevel"></div>
                <div class="inputWrapper">
                    <div @click="passwordShow = !passwordShow" class="showPassword" :class="{hidden: !passwordShow}" :title="!passwordShow ? titleOptions.show : titleOptions.hide"></div>
                    <input @blur="onBlur" :type="passwordShow === true ? 'text' : 'password'" id="newPassword" placeholder="Jelszó" :character="password" v-model="password" @keyup="showPasswordCounter">
                    <div @click="onCopyPass" class="copyPassword" :title="titleOptions.copy"></div>
                </div>
            </div>
            <div v-if="errorShow === true" class="formItem__alert">
                <div class="message">{{passwordLengthError}}</div>
                <ul v-if='passwordValidation.errors.length > 0 && !submitted' class=''>
                    <li v-for='error in passwordValidation.errors'>{{error}}</li>
                </ul>
            </div>
            <div class="formItem__bottom">
                <div class="passwordCounter">{{password.length}}
                    <transition name="slide-fade">
                        <span v-if="counterShow"></span>
                    </transition>
                </div>
                <div class="btnWrapper">
                    <div @click="onClear" class="btn--delPassword" :title="titleOptions.del"></div>
                    <div @click="onGenerate" class="btn--generatePassword">Generálás</div>
                </div>
            </div>
        </div>
    </div>
</template>

<script>

export default {
    name: 'App',
    data() {
        return {
            passwordShow: false,
            counterShow: false,
            errorShow: false,
            titleOptions: {
                show: 'Jelszó megjelenítése',
                hide: 'Jelszó elrejtése',
                del: 'Jelszó törlése',
                copy: 'Jelszó másolása'
            },
            password: '',
            checkPassword:'',
            submitted: false,
            // Set > Password length
            passwordLength: [
                {
                    minLength: 8,
                    errorMessage: 'A jelszó nem éri el a minimális karakter számot!'
                }
            ],
            // Set > Password rules
            rules: [
                {
                    name: "kisbetű",
                    character: "abcdefghijklmnopqrstuvwxyz",
                    checked: true,
                    errorMessage: 'Nem tartalmaz kisbetűt!',
                    regex: /[a-z]/
                },
                {
                    name: "nagybetű",
                    character: "ABCDEFGHIJKLMNOPQRSTUVWXYZ",
                    checked: true,
                    errorMessage: 'Nem tartalmaz nagybetűt!',
                    regex:/[A-Z]/
                },
                {
                    name: "szám",
                    character: "0123456789",
                    checked: true,
                    errorMessage: 'Nem tartalmaz számot!',
                    regex:/\d/
                },
                {
                    name: "speciális karakter",
                    character: "_-+=*&^%$#@!`[]{}()<>~|",
                    checked: false,
                    errorMessage: 'Nem tartalmaz speciális karekter!',
                    regex: /\W/
                }
            ]
        }
    },
    methods: {
        // Validate
        onBlur() {
            this.errorShow = true;
        },
        // Generate Password
        onGenerate: function () {
            // Random karakter választó egy adott string-ből
            String.prototype.pick = function(min, max) {
                var n, chars = '';
                if (typeof max === 'undefined') {
                    n = min;
                } else {
                    n = min + Math.floor(Math.random() * (max - min));
                }
                for (var i = 0; i < n; i++) {
                    chars += this.charAt(Math.floor(Math.random() * this.length));
                }
                return chars;
            };
            // String karaktereinek összekeverése
            String.prototype.shuffle = function() {
                var array = this.split('');
                var tmp, current, top = array.length;
                if (top) while (--top) {
                    current = Math.floor(Math.random() * (top + 1));
                    tmp = array[current];
                    array[current] = array[top];
                    array[top] = tmp;
                }
                return array.join('');
            };

            let result = "";
            let allowedCharacter = "";
            for (var j = 0; j < this.rules.length; j++) {
                if (this.rules[j].checked) {
                    allowedCharacter += this.rules[j].character;
                    result += this.rules[j].character.pick(1);
                }
            }

            let currentResultLength = result.length;
            result += allowedCharacter.pick(this.passwordLength[0].minLength - currentResultLength);

            this.password = result.shuffle();
            this.passwordShow = true;
        },
        // Clear Input Value
        onClear: function () {
            this.password = '';
            this.passwordShow = false;
            this.errorShow = false;
        },
        // Copy Password
        onCopyPass: function() {
            let passwordToCopy = this.password;
            try {
                // 1) Copy text
                navigator.clipboard.writeText(passwordToCopy);
                // 2) Catch errors
            } catch (err) {
                console.error('Jelszó másolása sikertelen:', err);
            }
        },
        showPasswordCounter() {
            if (this.password.length > 0) {
                this.counterShow = true;
            } else {
                this.counterShow = false;
            }
        }
    },
    computed: {
        passwordLengthError: function () {
            if (this.password.length < this.passwordLength[0].minLength) {
                return(this.passwordLength[0].errorMessage);
            }
        },
        // Password validation
        passwordValidation () {
            let errors = [];
            for (let condition of this.rules) {
                if (!condition.regex.test(this.password) && condition.checked) {
                    errors.push(condition.errorMessage)
                }
            }
            if (errors.length === 0) {
                return { valid:true, errors }
            } else {
                return { valid:false, errors }
            }
        },
        // Generate password note text
        noteText: function () {
            let characterText = "";
            for (var j = 0; j < this.rules.length; j++) {
                if (this.rules[j].checked) {
                    characterText += (this.rules[j].name + ', ');
                }
            }
            return ('Jelszó hossza: min. ' + this.passwordLength[0].minLength + ' karakter. Tartalmaz: ' + characterText.slice(0, -2) + '.');
        },
        // Password level
        scorePassword() {
            let score = 0;
            if (this.password === '') return score;

            let letters = {};
            for (let i = 0; i < this.password.length; i++) {
                letters[this.password[i]] = (letters[this.password[i]] || 0) + 1;
                score += 5.0 / letters[this.password[i]];
            }

            let variations = {
                digits: /\d/.test(this.password),
                lower: /[a-z]/.test(this.password),
                upper: /[A-Z]/.test(this.password),
                special: /\W/.test(this.password),
            };

            let variationCount = 0;
            for (let check in variations) {
                variationCount += (variations[check] == true) ? 1 : 0;
            }

            score += (variationCount - 1) * 10;

            return parseInt(score);
        },
        strengthLevel() {
            let pass = this.scorePassword;
            if(pass === 0) return 0;
            if(pass < 20) return 1;
            if(pass < 40) return 2;
            if(pass < 60) return 3;
            if(pass < 80) return 4;
            if(pass >= 80) return 5;
        }
    },
    components: {

    }
}
</script>

<style>
    .slide-fade-enter-active {
        transition: all .3s ease;
    }
    .slide-fade-leave-active {
        transition: all .125s ease-out;
    }
    .slide-fade-enter, .slide-fade-leave-to {
        transform: translateX(10px);
        opacity: 0;
    }
</style>
