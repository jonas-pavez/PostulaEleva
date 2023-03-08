<template>
  <div class="welcome-component">
    <h2>Ejemplo y bienvenida</h2>
    <p>Con este componente puedes ver cómo se pasa {{ algo }} de un componente a otro
       a través de los props. </p>
    <p>Y además hace una llamada a la api al crearse, que debería responderte con el
      saludo de bienvenida:</p>
    <p class="hi">{{ saludo }}</p>
  </div>
</template>

<script>
export default {
  name: 'WelcomeComponent',
  props: ['algo'],
  data() {
    return {
      saludo: '...Nada todavía',
    };
  },
  methods: {
    getWelcomeMessage() {
      // Acá tienes un ejemplo de llamada http a la api
      // puedes encontrar documentación para usar el cliente http de vue aquí:
      // https://github.com/pagekit/vue-resource
      this.$http.get('https://prueba-hamburguesas.herokuapp.com/test')
        .then((response) => { this.saludo = response.data; }, err => console.log(err));
    },
  },
  created() {
    console.log('componente welcome creado!, llamando a api');
    this.getWelcomeMessage();
  },
}
</script>

<style>
.welcome-component {
  padding: 10px;
  border: black 2px solid;
}
.hi {
  font-weight: bold;
  font-size: 1.5rem;
}
</style>
