<template>
    <div>
        <div class="mt-12 mb-6 border-b pb-5">
            <h3 class="text-4xl font-semibold">
                Inicio
            </h3>
            <p class="mt-3 text-muted-foreground">
                Sistema Digital para Controle de Frequência de Alunos
            </p>
        </div>
        <div class="text-sm">
            Univesp - Engenharia da Computação / Ciência de Dados - Projeto Integrador 1 - 01/2024
        </div>
    </div>
</template>

<script lang="ts" setup>
    definePageMeta({ middleware: 'auth' })
</script>