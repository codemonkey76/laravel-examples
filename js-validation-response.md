# Validation Error Bag Manager

## Vue Usage
```
import Validator from './Validator'

data: ()=>({
    validator: new Validator
}),

axios.get('/my-route').then((response)=>{
    ...
})
.catch((error)=>{
    this.validator.sync(error.response)
})

this.validator.clear()
this.validator.all()
this.validator.has('myField')
this.validator.get('myField', ['Please enter a default value.', 'Also do something else.'])
this.validator.first('myField', 'Please enter a default value.')
this.validator.in(['email', 'password'])
this.validator.firstEntry
this.validator.isInvalid
this.validator.sync(error)
this.validator.setMessage('Invalid Request.')
this.validator.setErrors({
  myField: ["Min length 5 chars."]
})
```

## Validator Class
```
export default class Validator{
    constructor(){
        this.message = null
        this.exception = null
        this.messageBag = {}
    }

    /**
     * Make New Instance
     * @param data
     * @return {Validator}
     */
    make(data = {}){
        return new Validator(data)
    }

    /**
     * (Method) clearErrors
     * @return void
     */
    clear() {
        this.message = null
        this.exception = null
        this.messageBag = {}
    }

    /**
     * Sync State
     * @param data {*}
     * @return {Validator}
     */
    sync(data = {}) {
        if(data.message){
            this.setMessage(data.message)
        }
        if(data.errors){
            this.setErrors(data.errors)
        }
        if(data.exception){
            this.exception = data.exception
        }
        return this
    }

    /**
     * Set Response Message
     * @param message {String}
     */
    setMessage(message = null){
        this.message = message
        return this
    }

    /**
     * Clear Response Message
     */
    clearMessage(){
        this.message = null
        return this
    }

    /**
     * Set Errors
     * @param messageBag
     */
    setErrors(messageBag = {}){
        let messages = {}
        Object.keys(messageBag).map((field)=>{
            if(Array.isArray(messageBag[field])){
                //Strip Dot Syntax from Field Names in Messages (for nested array fields)
                messageBag[field] = messageBag[field].map((value)=> {
                    if(typeof value === 'string'){
                        return value.replace(
                            field.replace(/_/g, ' '),
                            field.replace(/[._]/g, ' ')
                        )
                    }
                    return value
                })
            }
        })
        this.messageBag = Object.assign({}, messageBag)
        return this
    }

    /**
     * (Method) Get First Error Entry
     * @return {String|null}
     */
    get firstEntry() {
        const errors = Object.values(this.messageBag).flat(2);
        return errors.length > 0 ? errors[0] : null
    }

    /**
     * In Invalid.
     * @return {boolean}
     */
    get isInvalid(){
        return Object.entries(this.messageBag).length > 1
    }

    /**
     * Put field Error message.
     * @param messageBag
     */
    put(field, error){
        if(Array.isArray(this.messageBag[field])){
            this.messageBag[field].push(error)
        }else{
            let errors = {}
            errors[field] = [error]
            this.setErrors(Object.assign({}, this.setMessageBag, errors))
        }
        return this
    }

    /**
     * Forget field Error message.
     * @param field {String}
     */
    forget(field){
        delete this.messageBag[field]
        this.messageBag = Object.assign({}, this.messageBag)
        return this
    }

    /**
     * Has Error for field.
     * @param field {String}
     * @return {boolean}
     */
    has(field = null) {
        return field ? this.messageBag.hasOwnProperty(field) : false
    }

    /**
     * Get Error for field.
     * @param field {String}
     * @param fallback {*}
     * @return {*}
     */
    get(field, fallback = []) {
        return this.messageBag.hasOwnProperty(field) ? this.messageBag[field] : (fallback ? fallback : null)
    }

    /**
     * Get first Error for field.
     * @param field {String}
     * @param fallback {*}
     * @return {String|null}
     */
    first(field = null, fallback = false) {
        return this.messageBag.hasOwnProperty(field) ? this.messageBag[field][0] : fallback ? fallback : null
    }

    /**
     * Has Errors in fields.
     * @param fields {Array}
     * @return {Boolean}
     */
    in(...fields) {
        let hasErrors = false
        fields.some((field) => {
            hasErrors = this.messageBag.hasOwnProperty(field)
            return hasErrors
        })
        return hasErrors
    }

    /**
     * Get All Errors.
     * @return {Object}
     */
    all() {
        return this.messageBag
    }
}
```
