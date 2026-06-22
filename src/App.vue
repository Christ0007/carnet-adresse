<script setup>
import { ref, computed } from 'vue'

const nom = ref('')
const email = ref('')
const telephone = ref('')

const recherche = ref('')
const contacts = ref([])

function ajouterContact() {
  if (!nom.value || !email.value || !telephone.value) return

  contacts.value.push({
    id: Date.now(),
    nom: nom.value,
    email: email.value,
    telephone: telephone.value
  })

  nom.value = ''
  email.value = ''
  telephone.value = ''
}

function supprimerContact(id) {
  contacts.value = contacts.value.filter(c => c.id !== id)
}

const contactsFiltres = computed(() => {
  return contacts.value.filter(c =>
    c.nom.toLowerCase().includes(recherche.value.toLowerCase())
  )
})
</script>

<template>
  <div class="container">

    <h1>Carnet d'adresses</h1>

    <!-- FORMULAIRE -->
    <div class="form">
      <input v-model="nom" placeholder="Nom" />
      <input v-model="email" placeholder="Email" />
      <input v-model="telephone" placeholder="Téléphone" />

      <button @click="ajouterContact">
        Ajouter
      </button>
    </div>

    <!-- RECHERCHE -->
    <input
      v-model="recherche"
      placeholder="Rechercher un contact..."
      class="search"
    />

    <!-- LISTE -->
    <div class="list">
      <div
        v-for="contact in contactsFiltres"
        :key="contact.id"
        class="card"
      >
        <p><strong>{{ contact.nom }}</strong></p>
        <p>{{ contact.email }}</p>
        <p>{{ contact.telephone }}</p>

        <button @click="supprimerContact(contact.id)">
          Supprimer
        </button>
      </div>
    </div>

  </div>
</template>

<style>
.container {
  max-width: 600px;
  margin: 40px auto;
  font-family: Arial;
}

h1 {
  text-align: center;
  margin-bottom: 20px;
}

.form {
  display: flex;
  flex-direction: column;
  gap: 10px;
  margin-bottom: 10px;
}

input {
  padding: 10px;
}

.search {
  width: 100%;
  padding: 10px;
  margin-bottom: 15px;
}

button {
  padding: 10px;
  background: #3b82f6;
  color: white;
  border: none;
  cursor: pointer;
}

.list {
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.card {
  border: 1px solid #ddd;
  padding: 10px;
}
</style>