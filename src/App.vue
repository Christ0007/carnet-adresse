  <script setup>
  import { ref, computed, inject, watch } from 'vue'
  import swal from 'sweetalert2'


  const nom = ref('')
  const email = ref('')
  const telephone = ref('')
  const recherche = ref('')



  const contacts = ref(JSON.parse(localStorage.getItem('contacts') || '[]'))

  const contactEnEdition = ref(null)
  const nomEdit = ref('')
  const emailEdit = ref('')
  const telephoneEdit = ref('')

  watch(contacts, (valeur) => {
    localStorage.setItem('contacts', JSON.stringify(valeur))
  }, { deep: true })

  function ajouterContact() {
    if (!nom.value || !email.value || !telephone.value) {
      swal.fire({
        icon: 'warning',
        title: 'Champs manquants',
        text: 'Veuillez remplir tous les champs.',
        confirmButtonText: 'OK'
      })
      return
    }

    const emailValide = /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email.value.trim())
    if (!emailValide) {
      swal.fire({
        icon: 'error',
        title: 'Email invalide',
        text: 'Le format attendu est exemple@domaine.com',
        confirmButtonText: 'Corriger'
      })
      return
    }

    const telValide = /^\d{10}$/.test(telephone.value.trim())
    if (!telValide) {
      swal.fire({
        icon: 'error',
        title: 'Téléphone invalide',
        text: 'Le format attendu est 0123456789 (8 chiffres).',
        confirmButtonText: 'Corriger'
      })
      return
    }

    contacts.value.push({
      id: Date.now(),
      nom: nom.value,
      email: email.value,
      telephone: telephone.value
    })

    nom.value = ''
    email.value = ''
    telephone.value = ''

    swal.fire({
      icon: 'success',
      title: 'Contact ajouté',
      timer: 1500,
      showConfirmButton: false
    })
  }

  async function supprimerContact(id) {
    const result = await swal.fire({
      icon: 'warning',
      title: `Supprimer ce contact ?`,
      text: 'Cette action est irréversible.',
      showCancelButton: true,
      confirmButtonText: 'Supprimer',
      cancelButtonText: 'Annuler',
      confirmButtonColor: '#e74c3c'
    })

    if (result.isConfirmed) {
      contacts.value = contacts.value.filter(c => c.id !== id)
      swal.fire({
        icon: 'success',
        title: 'Contact supprimé',
        timer: 1200,
        showConfirmButton: false
      })
    }
  }

  function ouvrirEdition(contact) {
    contactEnEdition.value = contact.id
    nomEdit.value = contact.nom
    emailEdit.value = contact.email
    telephoneEdit.value = contact.telephone
  }

  async function sauvegarderEdition() {
    if (!nomEdit.value || !emailEdit.value || !telephoneEdit.value) {
      swal.fire({ icon: 'warning', title: 'Champs manquants', text: 'Veuillez remplir tous les champs.', confirmButtonText: 'OK' })
      return
    }

    const emailValide = /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(emailEdit.value.trim())
    if (!emailValide) {
      swal.fire({ icon: 'error', title: 'Email invalide', text: 'Le format attendu est exemple@domaine.com', confirmButtonText: 'Corriger' })
      return
    }

    const telValide = /^\d{10}$/.test(telephoneEdit.value.trim())
    if (!telValide) {
      swal.fire({ icon: 'error', title: 'Téléphone invalide', text: 'Le format attendu est 0022901234567 (00229 + 8 chiffres).', confirmButtonText: 'Corriger' })
      return
    }

    const index = contacts.value.findIndex(c => c.id === contactEnEdition.value)
    if (index !== -1) {
      contacts.value[index] = {
        ...contacts.value[index],
        nom: nomEdit.value,
        email: emailEdit.value,
        telephone: telephoneEdit.value
      }
    }

    contactEnEdition.value = null
    swal.fire({ icon: 'success', title: 'Contact modifié', timer: 1500, showConfirmButton: false })
  }

  function annulerEdition() {
    contactEnEdition.value = null
  }

  const contactsFiltres = computed(() => {
    const terme = recherche.value.toLowerCase()
    return contacts.value.filter(c =>
      c.nom.toLowerCase().includes(terme) ||
      c.email.toLowerCase().includes(terme) ||
      c.telephone.includes(terme)
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
        <!-- Mode affichage normal -->
        <template v-if="contactEnEdition !== contact.id">
          <p><strong>{{ contact.nom }}</strong></p>
          <p>{{ contact.email }}</p>
          <p>{{ contact.telephone }}</p>
          <button @click="ouvrirEdition(contact)">Modifier</button>
          <button @click="supprimerContact(contact.id)">Supprimer</button>
        </template>

        <!-- Mode édition -->
        <template v-else>
          <input v-model="nomEdit" placeholder="Nom" />
          <input v-model="emailEdit" placeholder="Email" />
          <input v-model="telephoneEdit" placeholder="Téléphone" />
          <button @click="sauvegarderEdition">Sauvegarder</button>
          <button @click="annulerEdition">Annuler</button>
        </template>
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
    width: 96%;
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

  .card input{
    margin: 5px;
  }

  .card button{
      align-items: flex-start;
      margin-right: 10px;
  }
  </style>