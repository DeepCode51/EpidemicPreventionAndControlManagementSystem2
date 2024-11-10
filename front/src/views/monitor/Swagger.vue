<template>
    <IFrame :src="url"/>
</template>

<script>
    import IFrame from "../../components/IFrame";
    export default {
        name: "Swagger",
        components: {IFrame},
        data() {
            return{
                url: "http://localhost:8081/swagger-ui/index.html"
            }
        }
    }
</script>
