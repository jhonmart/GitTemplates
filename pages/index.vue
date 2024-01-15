<template>
  <section class="section">
    <b-loading :active="isLoading" />
    <b-button type="is-success" class="copy" expanded @click="generateCopy">Copiar Template</b-button>
    <form class="mt-3">
      <b-field :label="field.name" v-for="field of models[modelName]" :key="`field_${field.name}`">
        <section v-if="field.fields" class="is-columns pl-5 ml-5">
          <div v-for="childField of field.fields" class="is-column" :key="`field_${field.name}_${childField.name}`">
            <label for="" class="has-text-white">{{ childField.name }}</label>
            <div
              v-for="childItem, index of childField.values"
              class="is-flex py-1 is-gap-5"
              :key="`field_${field.name}_${childField.name}_${index}`"
            >
              <b-input
                v-for="childFieldItem of childField.fields"
                :key="`field_${field.name}_${childField.name}_${index}_${childFieldItem}`"
                v-model="childItem[childFieldItem]"
                :placeholder="childFieldItem"
                :type="childField.type || 'textarea'"
                class="is-flex-1"
              />
              <b-button
                v-if="childField.many && index === childField.values.length - 1"
                type="is-success"
                class="ml-1"
                @click="childField.values.push({texto: '', link: ''})"
              >
                <b-icon icon="plus" />
              </b-button>
              <b-button v-else type="is-warning" class="ml-1" @click="childField.values.splice(index, 1)">
                <b-icon icon="minus" />
              </b-button>
            </div>
            <b-input v-if="!childField.values" :type="childField.type || 'textarea'" v-model="childField.value"
              class="is-flex-1" />
          </div>
        </section>
        <b-input v-else :type="field.type || 'textarea'" v-model="field.value" />
      </b-field>
    </form>
  </section>
</template>

<script>
export default {
  name: 'IndexPage',
  data() {
    return {
      modelName: "Default",
      models: {
        "Default": [
          {
            name: "Descrição",
            value: "",
            default: "DESCRIÇÃO_AQUI"
          },
          {
            name: "Cenário atual",
            icon: "💻",
            value: "",
            default: "CENÁRIO_ATUAL"
          },
          {
            name: "História",
            fields: [
              {
                name: "COMO",
                icon: "🙇",
                value: "",
                default: "____"
              },
              {
                name: "EU QUERO",
                icon: "🙋",
                value: "",
                default: "____"
              },
              {
                name: "PARA",
                icon: "🙆",
                value: "",
                default: "____"
              },
            ]
          },
          {
            name: "Redmine",
            icon: "📊",
            type: "text",
            value: "",
            default: "LINK_REDMINE"
          },
          {
            name: "Protótipo",
            fields: [
              {
                name: "Link",
                icon: "🎨",
                type: "text",
                fields: ["texto", "link"],
                many: true,
                values: [{texto: "", link: ""}],
                default: "LINK_DO_PROTOTIPO"
              }
            ]
          },
          {
            name: "Documentação",
            fields: [
              {
                name: "Link",
                icon: "📖",
                type: "text",
                fields: ["texto", "link"],
                many: true,
                values: [{texto: "", link: ""}],
                default: "LINK_DA_DOCUMENTACAO"
              }
            ]
          },
          {
            name: "Criterios",
            fields: [
              {
                name: "Ponto",
                icon: "<b>✔️RF@{index%2}:</b>",
                fields: ["texto"],
                many: true,
                values: [{texto: ""}],
                default: "CRITERIOS_DE_ACEITE"
              }
            ]
          },
          {
            name: "Observações",
            fields: [
              {
                name: "Ponto",
                icon: "⚠️",
                fields: ["texto"],
                many: true,
                values: [{texto: ""}],
                default: "Nesta seção escreva os itens de observação"
              }
            ]
          },
          {
            name: "Pesos",
            fields: [
              {
                name: "⌛ ~\"Back-end\":",
                value: -1,
                type: "number"
              },
              {
                name: "⌛ ~\"Front-end\":",
                value: -1,
                type: "number"
              }
            ]
          },
        ]
      },
      isLoading: false
    }
  },
  methods: {
    padronizeIcon(text, obj) {
      const find = /@{(?<param>\w+)%(?<size>\d+)}/.exec(text);
      if (find) {
        const { param, size } = find.groups;
        return text.replace(find[0], (Number(obj[param]) + 1).toString().padStart(size, "0"));
      }
      else return text;
    },
    generateText() {
      let base = `| Seção | Conteúdo |\n| ------ | ------ |\n`;
      for (const field of this.models[this.modelName]) {
        base += `| **${field.name}**: | `; // Title
        if (field.fields) {
          let childValue = "";
          for (const childField of field.fields) {
            if (childField.many) {
              for (const index in childField.values) {
                const childItem = childField.values[index];
                let childItemValue = `${this.padronizeIcon(childField.icon, { index }) || ""} `;
                if (childItem.link) childItemValue += `[${childItem.texto}](${childItem.link})`;
                else childItemValue += childItem.texto || childField.default;
                childValue += `${childItemValue}<br>`;
              }
            } else {
              let value = childField.value * 1;
              if (Boolean(value)) {
                if (value === -1) value = `Sem peso`;
                else if (value === -2) value = `~"spike"`;
              }
              childValue += `${childField.icon || ""} **${childField.name}** ${value || childField.value || childField.default}<br>`.replaceAll("\n", " ");
            }
          }

          base += `${childValue || field.default}`.replaceAll("\n", " ") + `|\n`;
        } else {
          if (field.icon) base += `${field.icon} `;
          base += `${field.value || field.default}`.replaceAll("\n", " ") + `|\n`;
        }
      };
      return base;
    },
    async generateCopy() {
      this.isLoading = true;
      const templateText = this.generateText();
      await navigator.clipboard.writeText(templateText);
      this.isLoading = false;
      this.$buefy.toast.open("Template copiado com sucesso");
    }
  }
}
</script>

<style>
.label {
  color: white;
}

textarea,
input {
  background-color: gray !important;
  color: white !important;
}
::placeholder {
  color: rgb(204, 204, 204) !important;
}

.copy {
  position: fixed;
  top: 0;
  left: 0;
  border-radius: 0;
  z-index: 10;
}

.is-flex-1 {
  flex: 1;
}
.is-gap-5 {
  gap: 5px;
}
</style>