<template>
    <!-- Contact Section -->
    <div class="container-fluid p-3" id="contact">
        <div class="row justify-content-center">
            <div class="col-lg-12 text-center">
                <h2 class="m-3" id="contact-title">CONTACT</h2>
            </div>

            <!-- Map Section Start -->  
<!--            <div class="container-fluid text-center"> -->
                <div class="row m-5 bg-white " >
                    <div class="col-md-6 col-lg-6 p-3 d-flex justify-content-center " id="map">
                        <iframe src="https://www.google.com/maps/embed?pb=!1m18!1m12!1m3!1d61829.271414904295!2d120.98985634505573!3d14.408161530094308!2m3!1f0!2f0!3f0!3m2!1i1024!2i768!4f13.1!3m3!1m2!1s0x3397d02cc7b1e345%3A0x51fdbee47cdaf013!2sMuntinlupa%2C%20Metro%20Manila!5e0!3m2!1sen!2sph!4v1751459752406!5m2!1sen!2sph" width="100%" height="450" allowfullscreen="" loading="lazy" referrerpolicy="no-referrer-when-downgrade"></iframe>
                    </div>
            
                    <!-- Contact Form -->
                    <div class="col-md-6 col-lg-6 p-3" id="contact-form">                   
                        <form class="px-5" @submit.prevent = "submitForm">
                            
                            <div class="mb-3">
                                <input type="text" v-model="name" class="form-control" placeholder="First Name MI Last name" required autocomplete="on">
                            </div>

                            <div class="mb-3">
                                <input type="email" v-model="email" class="form-control" placeholder="Email" required autocomplete="on">
                            </div>

                            <div class="mb-3">
                                <textarea v-model="message" class="form-control" rows="4" placeholder="Message" required autocomplete="on"></textarea>
                            </div>


                                <div class="d-flex justify-content-between align-items-center">
                                    <div class="d-flex gap-1">
                                        <a href="https://linkedin.com" target="_blank">
                                            <i class="fa-brands fa-linkedin fa-2xl icon-color"></i>
                                        </a>

                                        <a href="https://gitlab.com" target="_blank">
                                            <i class="fa-brands fa-gitlab fa-2xl icon-color"></i>
                                        </a>

                                        <a href="https://github.com" target="_blank">
                                            <i class="fa-brands fa-github fa-2xl icon-color" ></i>
                                        </a>
                                    </div>

                                    <div>
                                        <button type="button" class="d-flex ms-auto btn btn-submit rounded-pill px-4 justify-content-center w-50" data-bs-toggle="modal"data-bs-target="#submitModal" id="button">Submit</button>
                                    </div>

                                    
                                </div>  

                                 <!-- reCaptcha Checkbox -->
                            <div class="d-flex justify-content-end mt-2">
                                <div ref="recaptchaContainer"></div>
                            </div>

                        </form>     
                    </div>  
                        
                    </div>
                </div>          
            </div>
        <!-- End of Contact Section -->

        <!-- Submit Modal -->
        <div class="modal fade" id="submitModal" tabindex="-1" aria-labelledby="submitModalLabel" aria-hidden="true">
            <div class="modal-dialog">
                <div class="modal-content">

                    <!-- Modal Header -->
                    <div class="modal-header">
                        <h5 class="modal-title" id="submitModalLabel">I Got Your Message — Talk Soon!</h5>
                        <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close"></button>
                    </div>
                    <!-- Modal Body -->
                    <div class="modal-body">
                        Looking forward to collaborating with you and creating meaningful digital experiences.
                    </div>
                    <!-- Modal Footer -->
                    <div class="modal-footer">
                        <button type="button" class="btn btn-secondary" data-bs-dismiss="modal">Close</button>
                    </div>
                </div>
            </div>
        </div>

</template>

<script setup>
    import { ref, onMounted, onBeforeUnmount } from 'vue';
    import { Notyf } from 'notyf';
    import 'notyf/notyf.min.css'

    const notyf  = new Notyf();

    const WEB3FORMS_ACCESS_KEY = "ed9c6669-c88c-44a0-80e5-603d811e4de7";

    const subject = "New message from Portfolio Contact Form";

    const name = ref("")
    const email = ref("")
    const message = ref("")

    const isLoading = ref(false);

    const submitForm = async () => {

        if(!recaptchaToken.value) {
            notyf.error('Please verify that you are not a robot.')
            return;
        }


        isLoading.value = true;
        try {
            const response = await fetch("https://api.web3forms.com/submit", {
                method: "POST",
                headers: {
                    "Content-Type": "application/json",
                    Accept: "application/json",
                },
                body: JSON.stringify({
                    access_key: WEB3FORMS_ACCESS_KEY,
                    subject: subject,
                    name: name.value,
                    email: email.value,
                    message: message.value
                }),
            });
            const result = await response.json();

            if (result.success){
                console.log(result);
                isLoading.value = false;
                notyf.success("Message Sent!")
            }
        } catch (error) {
            console.log(error)
            isLoading.value = false;
            notyf.error("Failed to send message");
        } finally {
            resetRecaptcha()
        }
    }


    const SITE_KEY="6LfQBNwrAAAAAPqanWd4NI0r08yq8oX3Sz08zWQt"
    
    const recaptchaContainer = ref(null);
    const recaptchaWidgetId = ref(null);
    const recaptchaToken = ref('');

    function onRecaptchaSuccess(token) {
        recaptchaToken.value = token;
    }

    function onRecaptchaExpired() {
        recaptchaToken.value = "";
    }

    function renderRecaptcha() {
        if(!window.grecaptcha) {
            console.error('reCAPTCHA not loaded');
            return
        }

        recaptchaWidgetId.value = window.grecaptcha.render(
            recaptchaContainer.value, {
                sitekey: SITE_KEY,
                size: 'normal',
                callback: onRecaptchaSuccess,
                'expired-callback': onRecaptchaExpired
            })
    }

    function resetRecaptcha() {
        if (recaptchaWidgetId.value != null) {
            window.grecaptcha.reset(recaptchaWidgetId.value)
            recaptchaToken.value = '';
        }
    }


    onMounted(() => {
        const interval = setInterval(() => {
            if(window.grecaptcha && window.grecaptcha.render) {
                renderRecaptcha();
                clearInterval(interval)
            }
        }, 100)

        onBeforeUnmount(() => {
            clearInterval(interval);
        })
    })
</script>