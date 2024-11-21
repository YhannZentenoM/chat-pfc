<script>
  import { Input, Label, Spinner, Button, Dropdown, DropdownItem } from "flowbite-svelte"
  import { appStatusInfo, setAppStatusError } from "../store"
  const { url, pages, id } = $appStatusInfo

  let answer = ""
  let loading = false

  const numOfImagesToShow = Math.min(pages, 4)
  const images = Array.from({ length: numOfImagesToShow }, (_, i) => {
    const page = i + 1
    return url
      .replace("/upload/", `/upload/w_400,h_540,c_fill,pg_${page}/`)
      .replace(".pdf", ".jpg")
  })

  const handleSubmit = async (event) => {
    event.preventDefault()

    console.log("buscando..")

    loading = true
    answer = ""
    const question = event.target.question.value
    const exp = event.target.exp.value

    const searchParams = new URLSearchParams()
    searchParams.append("id", exp)
    searchParams.append("question", question)

    try {
      const eventSource = new EventSource(`/api/ask?${searchParams.toString()}`)

      eventSource.onmessage = (event) => {
        // loading = false
        console.log(exp)
        const incomingData = JSON.parse(event.data)

        if (incomingData === "__END__") {
          eventSource.close()
          return
        }

        answer += incomingData
      }
    } catch (e) {
      setAppStatusError()
    } finally {
      loading = false
    }
  }
</script>

<!-- <div class="grid grid-cols-4 gap-2">
  {#each images as image}
    <img
      class="rounded w-full h-auto aspect-[400/540]"
      src={image}
      alt="PDF page"
    />
  {/each}
</div> -->

<form class="mt-8" on:submit={handleSubmit}>
  <Label for="exp" class="block my-2 mt-5">Selecciona un expediente</Label>
  <select id="exp" class="block w-full">
    <option value="">Seleccione</option>
    <option value="c1a098ffcb49079c8180b18c7b15229a">047-2018-TA-CCIA</option>
    <option value="r1w092ffcf49371c8180b18c7b15559a">083-2019-TA-CCIA</option>
  </select>
  <Label for="question" class="block my-2 mt-5">Deja aquí tu pregunta</Label>
  <Input id="question" required placeholder="¿De qué trata este documento?"
  ></Input>
</form>

{#if loading}
  <div class="mt-10 flex justify-center items-center flex-col gap-y-2">
    <Spinner />
    <p class="opacity-75">Esperando respuesta...</p>
  </div>
{/if}

{#if answer}
  <div class="mt-8">
    <p class="font-medium">Respuesta:</p>
    <p>{answer}</p>
  </div>
{/if}
