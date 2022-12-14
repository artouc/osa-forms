<template lang="pug">
form.osa-forms(v-if="type === 'form'" @submit.prevent="submit")
    pre {{ strategyState }}
    pre {{ modelValue }}
    //- pre {{ modelDataObject }}
    pre {{ combinedDataObject }}
    //- p {{ isActiveInput }} {{ disabled }}
    slot

.osa-forms(v-else-if="type === 'text'" :class="theme === 'mini' ? 'osa-forms__input' : 'osa-forms__input--mini'")
    .osa-forms__label(v-if="label !== ''") {{ label }}
    input(
        type="text"
        :name="name"
        :placeholder="placeholder"
        :value="specificDataValue"
        :disabled="disabled"
        @input="valueUpdated"
    )
    .osa-forms__error
        transition(name="osa-forms__error" v-for="i in rowStrategy")
            p(v-if="i.passed === false") {{ i.error }}

.osa-forms(v-else-if="type === 'view'" :class="theme === 'mini' ? 'osa-forms__input' : 'osa-forms__input--mini'")
    .osa-forms__label(v-if="label !== ''") {{ label }}
    input(
        type="text"
        :name="name"
        :placeholder="placeholder"
        :value="specificDataValue"
        disabled
        @input="valueUpdated"
    )
    .osa-forms__error
        transition(name="osa-forms__error" v-for="i in rowStrategy")
            p(v-if="i.passed === false") {{ i.error }}

.osa-forms(v-else-if="type === 'date'" :class="theme === 'mini' ? 'osa-forms__input' : 'osa-forms__input--mini'")
    .osa-forms__label(v-if="label !== ''") {{ label }}
    input(
        type="date"
        :name="name"
        :placeholder="placeholder"
        :value="specificDataValue"
        :disabled="disabled"
        @input="valueUpdated"
    )
    .osa-forms__error
        transition(name="osa-forms__error" v-for="i in rowStrategy")
            p(v-if="i.passed === false") {{ i.error }}

//- .osa-forms(v-else-if="type === 'view'" :class="theme === 'mini' ? 'osa-forms__input' : 'osa-forms__input--mini'")
//-     .osa-forms__label(v-if="label !== ''") {{ label }}
//-     input(
//-         type="text"
//-         :name="name"
//-         :placeholder="placeholder"
//-         :value="specificDataValue"
//-         disabled
//-         @input="valueUpdated"
//-     )
//-     .osa-forms__error
//-         transition(name="osa-forms__error" v-for="i in rowStrategy")
//-             p(v-if="i.passed === false") {{ i.error }}
.osa-forms.osa-forms__input(v-else-if="type === 'number'")
    .osa-forms__label(v-if="label !== ''") {{ label }}
    input(
        type="number"
        :name="name"
        :placeholder="placeholder"
        :value="specificDataValue"
        @input="valueUpdated"
    )
    .osa-forms__error
        transition(name="osa-forms__error" v-for="i in rowStrategy")
            p(v-if="i.passed === false") {{ i.error }}


.osa-forms.osa-forms__select(v-else-if="type === 'select'")
    .osa-forms__label {{ label }}
    select(
        :name="name"
        :value="specificDataValue"
        :disabled="disabled"
        @input="valueUpdated"
    )
        option(v-if="placeholder" selected disabled) {{ placeholder }}
        template(v-for="item in options" :key="name")
            option(:value="item[0]") {{ item[1] }}
    .osa-forms__error
        transition(name="osa-forms__error" v-for="i in rowStrategy")
            p(v-if="i.passed === false") {{ i.error }}

template(v-if="type === 'submit'")
    button.c-button--max(:class="passedAllStrategyFlg.value ? 'osa-forms__submit' : 'osa-forms__submit--disable'" type="submit" v-if="theme === ''") {{ label }}
    button.c-button(:class="passedAllStrategyFlg.value ? 'osa-forms__submit' : 'osa-forms__submit--disable'" type="submit" v-else-if="theme === 'mini'") {{ label }}

</template>

<script>
// エラー用クラス作成
class SysFormError extends Error {}

export default {
    provide () {
        return {
            // eslint-disable-next-line
            passedAllStrategyFlg: computed(() => this.isPassedAllStrategies),
            modelDataObject: computed(() => this.combinedDataObject)
        }
    },
    inject: {
        passedAllStrategyFlg: {
            type: Boolean,
            default: null,
            required: false
        },
        modelDataObject: {
            type: Object,
            default: () => ({}),
            required: false
        }
    },
    // 各Props設定
    props: {
        // フォームのtype。textやselectなど
        type: {
            type: String,
            default: 'text'
        },
        label: {
            type: String,
            default: ''
        },
        name: {
            type: String,
            default: ''
        },
        placeholder: {
            type: String,
            default: ''
        },
        disabled: {
            type: Boolean,
            default: false
        },
        // selectだった場合のoptions
        options: {
            type: Array,
            default: () => ([])
        },
        strategy: {
            type: Array,
            default: () => ([
                {
                    'rule': 'none',
                    'message': ''
                }
            ])
        },
        // v-modelでバインドされたデータを扱うvueデフォルトの名前
        modelValue: {
            type: Object,
            default: () => ({})
        },
        // suffix: {
        //     type: Object,
        //     default: null
        // },
        theme: {
            type: String,
            default: ''
        }
    },
    // v-modelでバインドされたデータにemitするためのemits
    emits: ['update:modelValue', 'submit'],
    data () {
        return {
            // このSFCが子要素として使われた際にインプットデータを保存しておく
            specificDataKey: this.name,
            specificDataValue: '',
            // このSFCが親要素として使われた際にフォームデータを保存しておくデータオブジェクト
            combinedDataObject: {},
            // propsで受け取ったstrategyはRegExpに変換されていない、また、柔軟に変更できないためこのデータにパースする。
            rowStrategy: [],
            // フォーム内のバリデーション通過状況を保存しておくオブジェクト
            strategyState: {},
            // コンポーネント内のバリデーションを全て通過したフラグ
            isPassedAllStrategies: null
        }
    },
    watch: {
        // v-modelでバインドされたデータにemitする
        combinedDataObject: {
            handler (data) {
                this.$emit('update:modelValue', data)
            },
            deep: true
        },
        strategyState: {
            handler (data) {
                let i = Object.values(data).every((i) => {
                    return i
                })
                this.isPassedAllStrategies = i
            },
            deep: true
        },
        modelValue: {
            handler (data) {
                this.combinedDataObject = data
            },
            deep: true
        },
        modelDataObject: {
            handler (data) {
                this.specificDataValue = data.value[this.specificDataKey]
                // はじめからセットされていたデータでサブミット可能か検証
                // こいつがないとマイページの編集とかがうまく動かないけど、こいつがあるとデータが来る前にパってエラーがでちゃう
                this.validator(this.specificDataValue)
            },
            deep: true
        }
    },
    mounted () {
        nextTick( () => {
            // ストラテジーエラーを定義
            let invalidStrategyRuleError = (name) => { return new SysFormError(`[sys-form] Invalid strategy rule error: ${name}`) }

            // インプットでない場合ストラテジー関連の処理をスキップ
            if(this.type !== 'form' && this.type !== 'submit') {

                this.strategy.forEach(
                    // 第三引数にconvertedStrategyArrayをとり、処理中のArrayを格納している。
                    ({ rule, stress }, index, convertedStrategyArray) => {

                        // カスタムルール「require」をRegExpに変換。
                        if (rule === 'require') {
                            const requireRegExp = '^.+$'
                            convertedStrategyArray[index].rule = RegExp(requireRegExp)
                        }

                        // カスタムルール「none」をRegExpに変換。
                        else if (rule === 'none') {
                            const noneRegExp = '^.*$'
                            convertedStrategyArray[index].rule = RegExp(noneRegExp)
                        }

                        // カスタムルール「length」をRegExpに変換。
                        else if (rule.startsWith('length:')) {
                            if (RegExp(/^length:[0-9]+(~[0-9]+?)?$/).test(rule)) {
                                let ruleValue, lengthRuleMinValue, lengthRuleMaxValue, lengthRuleRegExp

                                ruleValue = rule.substring(rule.indexOf(':') + 1)
                                if (ruleValue.indexOf('~') !== -1) {
                                    lengthRuleMaxValue = ruleValue.substring(ruleValue.indexOf('~') + 1)
                                    lengthRuleMinValue = ruleValue.substring(0, ruleValue.indexOf('~'))
                                } else {
                                    lengthRuleMaxValue = ruleValue
                                    lengthRuleMinValue = 0
                                }
                                if (Number(lengthRuleMinValue) > Number(lengthRuleMaxValue)) {
                                    throw invalidStrategyRuleError('lengthの最大値は最小値以上である必要があります。')
                                } else {
                                    lengthRuleRegExp = `^.{${lengthRuleMinValue},${lengthRuleMaxValue}}$`
                                    convertedStrategyArray[index].rule = RegExp(lengthRuleRegExp)
                                }
                            } else {
                                throw invalidStrategyRuleError('`length:num`または`length:num~num`で指定してください。')
                            }
                        }

                        // カスタムルール「allow」をRegExpに変換。
                        else if (rule.startsWith('allow:')) {
                            let ruleValue, allowRuleTypes = [], allowRuleRegExp

                            ruleValue = rule.substring(rule.indexOf(':') + 1)
                            if (ruleValue.indexOf(',') !== -1) {
                                allowRuleTypes = ruleValue.split(',')
                            } else {
                                allowRuleTypes.push(ruleValue)
                            }
                            allowRuleRegExp = '^('
                            allowRuleTypes.forEach(
                                (type, index) => {
                                    if (index > 0) {
                                        allowRuleRegExp = allowRuleRegExp + '|'
                                    }
                                    switch (type) {
                                    case 'space':
                                        allowRuleRegExp = allowRuleRegExp + '\u3000| '
                                        break
                                    case 'num':
                                        allowRuleRegExp = allowRuleRegExp + '[0-9]'
                                        break
                                    case 'uppercase':
                                        allowRuleRegExp = allowRuleRegExp + '[A-Z]'
                                        break
                                    case 'lowercase':
                                        allowRuleRegExp = allowRuleRegExp + '[a-z]'
                                        break
                                    default:
                                        throw invalidStrategyRuleError(`タイプ「${type}」はサポートされていません。`)
                                    }
                                }
                            )
                            allowRuleRegExp = allowRuleRegExp + ')*$'
                            convertedStrategyArray[index].rule = RegExp(allowRuleRegExp)
                        }

                        // カスタムルール「disallow」をRegExpに変換。
                        else if (rule.startsWith('disallow:')) {
                            let ruleValue, disallowRuleTypes = [], disallowRuleRegExp

                            ruleValue = rule.substring(rule.indexOf(':') + 1)
                            if (ruleValue.indexOf(',') !== -1) {
                                disallowRuleTypes = ruleValue.split(',')
                            } else {
                                disallowRuleTypes.push(ruleValue)
                            }
                            disallowRuleRegExp = '^(?!.*('
                            disallowRuleTypes.forEach(
                                (type, index) => {
                                    if (index > 0) {
                                        disallowRuleRegExp = disallowRuleRegExp + '|'
                                    }
                                    switch (type) {
                                    case 'space':
                                        disallowRuleRegExp = disallowRuleRegExp + '\u3000| '
                                        break
                                    case 'num':
                                        disallowRuleRegExp = disallowRuleRegExp + '[0-9]'
                                        break
                                    case 'uppercase':
                                        disallowRuleRegExp = disallowRuleRegExp + '[A-Z]'
                                        break
                                    case 'lowercase':
                                        disallowRuleRegExp = disallowRuleRegExp + '[a-z]'
                                        break
                                    default:
                                        throw invalidStrategyRuleError(`タイプ「${type}」はサポートされていません。`)
                                    }
                                }
                            )
                            disallowRuleRegExp = disallowRuleRegExp + ')).*$'
                            convertedStrategyArray[index].rule = RegExp(disallowRuleRegExp)
                        }

                        // カスタムルール「custom」をRegExpに変換。
                        else if (rule.startsWith('custom:')) {
                            let ruleValue, customRuleRegExp

                            ruleValue = rule.substring(rule.indexOf(':') + 1)
                            customRuleRegExp = ruleValue
                            convertedStrategyArray[index].rule = RegExp(customRuleRegExp)
                        }

                        // 存在しないカスタムルールに対するエラー。
                        else {
                            throw invalidStrategyRuleError(`「${rule}」は存在しないルールです。`)
                        }

                        // ストラテジー「rule」「error」にくわえ「passed」列を追加
                        // 強調がオンの場合はじめからエラーを表示
                        if (stress) {
                            // 強調はオンだがはじめからデータがある場合は非表示
                            if (this.$parent.modelValue[`${this.specificDataKey}`]) {
                                return
                            } else {
                                convertedStrategyArray[index].passed = false
                            }
                        } else {
                            convertedStrategyArray[index].passed = null
                        }

                        // convertedStrategyArrayはforEach内の引数でしかないため、SFCのデータにパースする。
                        this.rowStrategy = convertedStrategyArray
                    }
                )

                // // マウント時にSubmit可能か検証する。
                // this.submitGate

                // v-modelで指定されたオブジェクトに既に値が存在していた際
                if (this.$parent.modelValue) {

                    // 既存のデータを破壊しないように再代入
                    this.$parent.combinedDataObject = this.$parent.modelValue

                    // インプットのnameと一致するkeyがあった場合、そのvalueをインプットのvalueに再代入
                    if (this.$parent.modelValue[this.specificDataKey]) {
                        this.specificDataValue = this.$parent.modelValue[this.specificDataKey]
                        // はじめからセットされていたデータでサブミット可能か検証
                        this.validator(this.specificDataValue)
                    }
                }

                // 親要素にcombinedDataObjectがある（このSFCが子として呼び出されている）場合に各データオブジェクトを追加
                if(this.$parent.combinedDataObject) {
                    this.$parent.combinedDataObject[this.specificDataKey] = this.specificDataValue
                }

                if(this.specificDataValue === '') {
                    return
                } else {
                    this.validator(this.specificDataValue)
                }
            }
        })
    },
    methods: {
        submit () {
            if (this.isPassedAllStrategies) {
                this.$emit('submit')
            }
        },
        valueUpdated (event) {
            // 値が変更された際インプットデータを反映
            this.specificDataValue = event.target.value

            // 親要素にcombinedDataObjectがある（このSFCが子として呼び出されている）場合に各データオブジェクトを変更
            if(this.$parent.combinedDataObject) {
                this.$parent.combinedDataObject[this.specificDataKey] = this.specificDataValue
            }

            // バリデーションを行う
            this.validator(event.target.value)
        },
        validator (value) {
            this.rowStrategy.forEach(
                ({rule}, index) => {
                    const result = rule.test(value)
                    this.rowStrategy[index].passed = result
                }
            )

            this.submitGate()
        },
        // バリデーションクリアによってSubmitボタンを解放する関数
        submitGate () {
            // 全てのstrategyのpassedを取得し、everyにかける。
            let specificStrategyState = []
            this.rowStrategy.forEach(
                ({passed}) => {
                    specificStrategyState.push(passed)
                }
            )
            // このコンポーネント内の全てのバリデーションを通過したらtrueとなる。
            const submitGateResult = specificStrategyState.every(i => i)

            // 親要素にcombinedDataObjectがある（このSFCが子として呼び出されている）場合に親のstrategyStateを更新
            if(this.$parent.combinedDataObject) {
                this.$parent.strategyState[this.name] = submitGateResult
            }
        }
    }
}
</script>