<template>

  <UiTabsContent value="Todos" class="mt-4">
    <UiDatatable @ready="initializeTable" :options="options" :columns="columns" :data="data">

      <template #actions="{ cellData }" class="flex items-center m-0 p-0">
        <UiButton variant="outline" @click="edit(cellData, $event)" class="px-2 m-0 mr-2 h-8">
          Editar
        </UiButton>

        <UiButton variant="destructive" @click="remove(cellData, $event)" class="px-2 m-0 ml-2 h-8">
          Deletar
        </UiButton>

      </template>

    </UiDatatable>
  </UiTabsContent>



  <AlertDialogRoot v-model:open="modalState">
    <AlertDialogPortal>
      <AlertDialogOverlay
        class="bg-background/80 backdrop-blur-sm data-[state=open]:animate-overlayShow fixed inset-0 z-30">

        <div class="absolute
                h-full
                w-full
                dark:bg-[radial-gradient(theme(colors.border)_1px,transparent_1px)]
                bg-[radial-gradient(theme(colors.border)_1px,transparent_1px)]
                [background-size:15px_15px]
                [mask-image:radial-gradient(ellipse_600px_600px_at_50%_50%,#000_10%,transparent_100%)]" />

      </AlertDialogOverlay>
      <AlertDialogContent
        class="z-[100] text-[15px] data-[state=open]:animate-contentShow fixed top-[50%] left-[50%] max-h-[85vh] w-[90vw] max-w-[700px] translate-x-[-50%] translate-y-[-50%] rounded-lg bg-primary-foreground dark:bg-black p-[25px] shadow-[0_0px_50px_-30px_rgba(0,0,0,0.5)] dark:shadow-[0_0px_80px_-50px_rgba(0,0,0,0.5)] dark:shadow-gray-500 sm:w-[700px] focus:outline-none border border-input  ">
        <AlertDialogTitle class="text-xl font-semibold mb-4">
          {{ isEditing ? 'Editar Professor' : 'Adicionar Novo Professor' }}
        </AlertDialogTitle>
        <AlertDialogDescription class="text-mauve11 mt-4 mb-5 text-[15px] leading-normal">
          <form @submit.prevent="handleSave">
            <div class="grid w-full items-center gap-4">
              <div class="flex flex-col space-y-1.5">

                <UiLabel for="nome" class=""> Nome </UiLabel>
                <input id="nome" v-model="newUser.nome" autocomplete="off" class="alert-input" />

                <UiLabel for="email"> Email </UiLabel>
                <input id="email" v-model="newUser.email" autocomplete="off" class="alert-input" />

                <UiLabel for="celular"> Celular </UiLabel>
                <input id="celular" v-model="newUser.celular" autocomplete="off" class="alert-input" />

              </div>
            </div>
          </form>
        </AlertDialogDescription>
        <div class="flex justify-end gap-[25px]">
          <AlertDialogCancel
            class="text-mauve11 bg-mauve4 hover:bg-mauve5 focus:shadow-mauve7 inline-flex h-[35px] items-center justify-center rounded-[4px] px-[15px] font-semibold leading-none outline-none focus:shadow-[0_0_0_2px]">
            Voltar
          </AlertDialogCancel>
          <AlertDialogAction
            class="text-red11 bg-red4 hover:bg-red5 focus:shadow-red7 inline-flex h-[35px] items-center justify-center rounded-[4px] px-[15px] font-semibold leading-none outline-none focus:shadow-[0_0_0_2px]"
            @click="handleSave()">
            {{ isEditing ? 'Atualizar' : 'Salvar' }}
          </AlertDialogAction>
        </div>
      </AlertDialogContent>
    </AlertDialogPortal>
  </AlertDialogRoot>
  <!-- Edit Button -->




</template>
<script lang="ts" setup>
  import { ref, shallowRef, watch, reactive } from 'vue';
  import type DataTableRef from 'datatables.net';
  import type { Config, ConfigColumns } from 'datatables.net';
  import languageBR from 'datatables.net-plugins/i18n/pt-BR.mjs';
  import {
    AlertDialogAction,
    AlertDialogCancel,
    AlertDialogContent,
    AlertDialogDescription,
    AlertDialogOverlay,
    AlertDialogPortal,
    AlertDialogRoot,
    AlertDialogTitle,
    AlertDialogTrigger,
  } from 'radix-vue';

  const selectedRows = ref(0);
  const isEditing = ref(false);
  const modalState = ref(false);
  const editingRowIndex = ref<number | null>(null);

  const newUser = reactive({
    id: null,
    nome: '',
    email: '',
    celular: '',
  });

  const data = await $fetch<any>("http://localhost:3000/api/professor");

  const tableRef = shallowRef<InstanceType<typeof DataTableRef<any[]>> | null>(null);

  const options: Config = {

    dom: "<'flex flex-col lg:flex-row w-full lg:items-start lg:justify-between gap-5 mb-5 lg:pr-1'Bf><'border rounded-lg't><'flex flex-col lg:flex-row gap-5 items-center lg:justify-between w-full pt-3 p-5 m-auto'lp>",
    searching: true,
    language: languageBR,
    paging: true,
    ordering: true,
    responsive: true,
    autoWidth: true,
    select: {
      style: "multi",
    },
    buttons: [
      {
        text: (selectedRows.value > 0 ? "Desselecionar" : "Selecionar Todos"),
        action: function (e, dt, node, config) {
          if (selectedRows.value > 0) {
            dt.rows().deselect();
            selectedRows.value = 0;
          } else {
            dt.rows().select();
            selectedRows.value = dt.rows({ selected: true }).count();
          }
        },
      },
      {
        text: "Novo",
        action: function (e, dt, node, config) {
          isEditing.value = false;
          Object.assign(newUser, { id: null, nome: '', email: '', celular: '' }); // Reset newUser object
          modalState.value = true;
        },
      }
    ],
  };
  const columns: ConfigColumns[] = [
    { data: "id", title: "Id" },
    { data: "nome", title: "Nome" },
    { data: "email", title: "Email" },
    { data: "celular", title: "Celular" },
    {
      data: null,
      title: "",
      className: "no-export",
      searchable: false,
      orderable: false,
      name: "actions",
      render: "#actions",
      responsivePriority: 3,

    }
  ];






  // ------------------------------------------------------------

  watch(selectedRows, (newValue) => {
    const selectButton = tableRef.value?.button(0); // Select/Deselect button index is 0
    if (selectButton) {
      selectButton.text(newValue > 0 ? 'Selecionar Nenhum' : 'Selecionar Todos');
    }
  });

  function initializeTable(instance: any) {
    tableRef.value = instance;
    tableRef.value?.on('select.dt deselect.dt', updateSelectedRowsCount);
    bindActionButtons();
  }


  function updateSelectedRowsCount() {
    selectedRows.value = tableRef.value?.rows({ selected: true }).count() || 0;
  }

  function bindActionButtons() {
    document.querySelectorAll('.edit-btn').forEach(button => {
      button.addEventListener('click', () => {
        const row = tableRef.value?.row(button).data();
        edit(row, event);
      });
    });

    document.querySelectorAll('.remove-btn').forEach(button => {
      button.addEventListener('click', () => {
        const row = tableRef.value?.row(button).data();
        remove(row, event);
      });
    });
  }

  watchEffect(() => {
    if (tableRef.value) {
      bindActionButtons();
    }
  });

  // ------------------------------------------------------------




  async function remove(user: any, event: Event) {
    event.stopPropagation(); // Impede a propagação do evento de clique

    $fetch("http://localhost:3000/api/professor", {
      method: "DELETE",
      headers: {
        "Content-Type": "application/json",
      },
      body: { user },
    }).then(() => {

      const row = tableRef.value?.row((idx, data) => data.id === user.id);
      if (row) {
        row.remove().draw(false);
      }
    });
  }


  function edit(user: any, event: Event) {
    event.stopPropagation(); // Impede a propagação do evento de clique
    isEditing.value = true;
    Object.assign(newUser, user); // Atualiza o objeto newUser com os dados do usuário
    editingRowIndex.value = data.findIndex((item: any) => item.id === user.id);
    modalState.value = true;
  }


  async function handleSave() {
    if (isEditing.value) {
      const response = await $fetch(`http://localhost:3000/api/professor`, {
        method: "PUT",
        headers: {
          "Content-Type": "application/json",
        },
        body: JSON.stringify(newUser),
      });

      const row = tableRef.value?.row(editingRowIndex.value);
      if (row) {
        row.data(response).draw(false);
        Object.assign(data[editingRowIndex.value], response); // Atualiza os dados na variável data
      }
    } else {
      const response = await $fetch("http://localhost:3000/api/professor", {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
        },
        body: JSON.stringify(newUser),
      });

      tableRef.value?.row.add(response).draw();
      data.push(response); // Adiciona o novo usuário aos dados
    }

    modalState.value = false;
    Object.assign(newUser, { id: null, nome: '', email: '', celular: '' }); // Reset newUser object
  }
</script>


<style scoped>
  .alert-input {
    @apply h-10 w-full rounded-md border border-input bg-background px-3 py-2 text-[16px] ring-offset-background file:border-0 file:bg-transparent file:text-sm file:font-medium file:text-foreground placeholder:text-muted-foreground file:hover:cursor-pointer focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-ring focus-visible:ring-offset-2 disabled:cursor-not-allowed disabled:opacity-50 sm:text-sm;
  }
</style>