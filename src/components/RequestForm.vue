<template lang="html">
  <div>
    <fieldset>
      <legend>Request</legend>
      <select v-model="method">
        <option>GET</option>
        <option>POST</option>
        <option>PUT</option>
        <option>DELETE</option>
        <option>OPTIONS</option>
      </select>
      <input v-model="url" />
      <input v-model="path" />
      <button @click="sendRequest">send</button>
    </fieldset>
    <fieldset>
      <legend>Authentication</legend>
      <label for="auth">Authentication Type</label>
      <select v-model="auth">
        <option>none</option>
        <option>Basic</option>
      </select>
      <section v-if="auth === 'Basic'">
        <label for="http_basic_user">User</label>
        <input v-model="httpUser" />
        <label for="http_basic_passwd">Password</label>
        <input v-model="httpPassword" type="password" />
      </section>
    </fieldset>
    <fieldset>
      <legend>Request Params</legend>
      <button @click="addRequestParam">add param</button>
      <ol>
        <li :key="param.key" v-for="(param, index) in params">
          <label :for="'param' + index">Key</label>
          <input :name="'param' + index" v-model="param.key" />
          <label :for="'value' + index">Value</label>
          <input :name="'value' + index" v-model="param.value" />
          <button @click="removeRequestParam(index)">remove</button>
        </li>
      </ol>
      <pre><code>{{queryString}}</code></pre>
    </fieldset>
    <fieldset v-if="method === 'POST' || method === 'PUT'">
      <legend>Request Body</legend>
      <section>
        <label>Content Type</label>
        <select v-model="contentType">
          <option>application/json</option>
          <option>www-form/urlencoded</option>
        </select>
        <button @click="addRequestBodyParam">add param</button>
      </section>
      <ol>
        <li :key="param.key" v-for="(param, index) in bodyParams">
          <label :for="'bparam' + index">Key</label>
          <input :name="'bparam' + index" v-model="param.key" />
          <label :for="'bvalue' + index">Value</label>
          <input :name="'bvalue' + index" v-model="param.value" />
          <button @click="removeRequestBodyParam(index)">remove</button>
        </li>
      </ol>
      <pre><code>{{rawRequestBody}}</code></pre>
    </fieldset>
    <fieldset class="api-response">
      <legend>Response</legend>
      <section>Status: {{ response.status }}</section>
      <table>
        <tr :key="key" v-for="(value, key) in response.headers">
          <td style="width: 20%;"><input :value="key" readonly /></td>
          <td><input :value="value" readonly /></td>
        </tr>
      </table>
      <textarea rows="5" readonly v-model="response.body"></textarea>
    </fieldset>
  </div>
</template>

<script>
import parseHeaders from "../helpers/parseHeaders.js";

export default {
  data() {
    return {
      method: "GET",
      url: "https://yesno.wtf",
      auth: "none",
      path: "/api",
      httpUser: "",
      httpPassword: "",
      params: [],
      bodyParams: [],
      contentType: "application/json",
      response: {
        status: "",
        headers: "",
        body: "",
      },
    };
  },
  computed: {
    rawRequestBody() {
      const { bodyParams } = this;
      if (this.contentType === "application/json") {
        try {
          const obj = JSON.parse(`{
            ${bodyParams
              .filter((p) => !!p.key)
              .map((p) => `"${p.key}": "${p.value}"`)
              .join()}
          }`);
          return JSON.stringify(obj);
        } catch (ex) {
          return "invalid";
        }
      } else {
        return bodyParams
          .filter((p) => !!p.key)
          .map((p) => p.key + "=" + encodeURIComponent(p.value))
          .join("&");
      }
    },
    queryString() {
      const result = this.params
        .filter((p) => !!p.key)
        .map((p) => p.key + "=" + encodeURIComponent(p.value))
        .join("&");
      return result === "" ? "" : "?" + result;
    },
  },
  methods: {
    sendRequest() {
      const xhr = new XMLHttpRequest();
      const user = this.auth === "Basic" ? this.httpUser : null;
      const pswd = this.auth === "Basic" ? this.httpPassword : null;
      xhr.open(
        this.method,
        this.url + this.path + this.queryString,
        true,
        user,
        pswd
      );
      if (this.method === "POST" || this.method === "PUT") {
        const requestBody = this.rawRequestBody;
        xhr.setRequestHeader("Content-Length", requestBody.length);
        xhr.setRequestHeader(
          "Content-Type",
          this.contentType + "; charset=utf-8"
        );
        xhr.send(requestBody);
      } else {
        xhr.send();
      }
      xhr.onload = () => {
        this.response.status = xhr.status;
        const headers = (this.response.headers = parseHeaders(xhr));
        if ((headers["content-type"] || "").startsWith("application/json")) {
          this.response.body = JSON.stringify(
            JSON.parse(xhr.responseText),
            null,
            2
          );
        } else {
          this.response.body = xhr.responseText;
        }
      };
    },
    addRequestParam() {
      this.params.push({ key: "", value: "" });
      return false;
    },
    removeRequestParam(index) {
      this.params.splice(index, 1);
    },
    addRequestBodyParam() {
      this.bodyParams.push({ key: "", value: "" });
      return false;
    },
    removeRequestBodyParam(index) {
      this.bodyParams.splice(index, 1);
    },
  },
};
</script>

<style lang="css">
fieldset {
  margin: 1em 0;
  border: 1px solid #ddd;
  border-radius: 2px;
  background: #eee;
}

legend {
  font-family: sans-serif;
  font-size: 80%;
  color: #fff;
  font-weight: bold;
  background: #49f;
  padding: 0.1em 0.5em;
  border-radius: 8px;
}

@media (max-width: 500px) {
  fieldset label {
    display: block;
  }

  fieldset input {
    display: block;
    width: 100%;
    box-sizing: border-box;
  }

  input + button,
  input + input,
  select + input {
    margin: 0.5em 0 0 0;
  }
}

fieldset textarea {
  width: 100%;
  box-sizing: border-box;
}

fieldset > section,
fieldset > table {
  margin: 0.5em 0;
}

fieldset > table {
  width: 100%;
  border-style: collaped;
  border: 1px solid #ccc;
}

fieldset table td input {
  width: 100%;
  box-sizing: border-box;
}

fieldset > ol {
  padding: 0 1em;
}

fieldset > ol > li {
  margin: 0.5em 0;
}

fieldset.api-response > legend {
  background: #f72;
}
</style>
