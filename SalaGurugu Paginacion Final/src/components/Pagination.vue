<template>
  <nav aria-label="Page navigation example">
    <ul class="pagination">
      <li class="page-item">
        <router-link class="page-link" to="">Anterior</router-link>
      </li>
      <li class="page-item">
        <router-link class="page-link" to="">1</router-link>
      </li>
    </ul>
  </nav>
</template>

<script>
export default {
    props: {
    siguiente: String,
  }
}
</script>

