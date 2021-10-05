<template>
  <div class="container content mt-3">
    <h1>Apuração Mensal de IRPF</h1>
    <div v-if="json">
      <table class="table table-sm">
        <thead class="table-dark">
          <tr>
            <th rowspan="2" class="title">Mês</th>
            <th rowspan="2" class="title">Data</th>
            <th rowspan="2" class="title">Ativo</th>
            <th colspan="5" class="title">Lucro</th>
            <th colspan="4" class="title">Vendas</th>
            <th colspan="4" class="title">Prejuízo Acum.</th>
            <th colspan="7" class="title">Imposto</th>
          </tr>
          <tr>
            <th>Operação</th>
            <th>BDR</th>
            <th>ETF</th>
            <th>Ação</th>
            <th>FII</th>
            <th>BDR</th>
            <th>ETF</th>
            <th>Ação</th>
            <th>FII</th>
            <th>BDR</th>
            <th>ETF</th>
            <th>Ação</th>
            <th>FII</th>
            <th>BDR</th>
            <th>ETF</th>
            <th>Ação</th>
            <th>FII</th>
            <th>Devido</th>
            <th>Acum.</th>
            <th>A Pagar</th>
          </tr>
        </thead>

        <tbody>
          <tr v-for="t in tabelaDeVendas" :key="t.id">
            <td v-if="t.rowSpan" :rowspan="t.rowSpan" class="title">
              {{ t._mes }}
            </td>
            <td class="title">{{ t.data }}</td>
            <td class="title">{{ t.ticker }}</td>
            <td class="bg-light">
              {{ formatPrice(t._lucroTotal) }}
            </td>
            <td v-if="t.rowSpan" :rowspan="t.rowSpan" class="bg-light">
              {{ formatPrice(t._bdrLucro) }}
            </td>
            <td v-if="t.rowSpan" :rowspan="t.rowSpan" class="bg-light">
              {{ formatPrice(t._etfLucro) }}
            </td>
            <td v-if="t.rowSpan" :rowspan="t.rowSpan" class="bg-light">
              {{ formatPrice(t._acaoLucro) }}
            </td>
            <td v-if="t.rowSpan" :rowspan="t.rowSpan" class="bg-light">
              {{ formatPrice(t._fiiLucro) }}
            </td>
            <td v-if="t.rowSpan" :rowspan="t.rowSpan">
              {{ formatPrice(t._bdrVendas) }}
            </td>
            <td v-if="t.rowSpan" :rowspan="t.rowSpan">
              {{ formatPrice(t._etfVendas) }}
            </td>
            <td v-if="t.rowSpan" :rowspan="t.rowSpan">
              {{ formatPrice(t._acaoVendas) }}
            </td>
            <td v-if="t.rowSpan" :rowspan="t.rowSpan">
              {{ formatPrice(t._fiiVendas) }}
            </td>
            <td v-if="t.rowSpan" :rowspan="t.rowSpan" class="bg-light">
              {{ formatPrice(t._bdrPrejuizoAcum) }}
            </td>
            <td v-if="t.rowSpan" :rowspan="t.rowSpan" class="bg-light">
              {{ formatPrice(t._etfPrejuizoAcum) }}
            </td>
            <td v-if="t.rowSpan" :rowspan="t.rowSpan" class="bg-light">
              {{ formatPrice(t._acaoPrejuizoAcum) }}
            </td>
            <td v-if="t.rowSpan" :rowspan="t.rowSpan" class="bg-light">
              {{ formatPrice(t._fiiPrejuizoAcum) }}
            </td>
            <td v-if="t.rowSpan" :rowspan="t.rowSpan">
              {{ formatPrice(t._bdrImposto) }}
            </td>
            <td v-if="t.rowSpan" :rowspan="t.rowSpan">
              {{ formatPrice(t._etfImposto) }}
            </td>
            <td v-if="t.rowSpan" :rowspan="t.rowSpan">
              {{ formatPrice(t._acaoImposto) }}
            </td>
            <td v-if="t.rowSpan" :rowspan="t.rowSpan">
              {{ formatPrice(t._fiiImposto) }}
            </td>
            <td v-if="t.rowSpan" :rowspan="t.rowSpan">
              {{ formatPrice(t._impostoDevido) }}
            </td>
            <td v-if="t.rowSpan" :rowspan="t.rowSpan">
              {{ formatPrice(t._impostoAcum) }}
            </td>
            <td v-if="t.rowSpan" :rowspan="t.rowSpan">
              {{ formatPrice(t._impostoAPagar) }}
            </td>
          </tr>
        </tbody>
      </table>
      <p class="text-muted">
        Confira os cálculos antes de fazer sua declaração pois esse aplicativo é
        <b>experimental</b>.
      </p>
    </div>
    <form>
      <label>Transações</label>
      <b-textarea v-model="json" />
      <p class="text-muted">
        No Chrome, abra Inspecionar/Rede, acesse o site Real Valor, naveque para
        o extrato, copie a Resposta de "transacoes" e cole o JSON aqui.
      </p>
    </form>
  </div>
</template>

<script>
export default {
  data() {
    return {
      json: undefined,
    }
  },
  computed: {
    realValorTransacoes() {
      if (!this.json) return []
      return JSON.parse(this.json).data.transacoes
    },
    transacoes() {
      const l = []
      const map = {}
      const self = this
      const fix = function (transacao) {
        const t = { ...transacao }
        if (!map[t.ticker])
          map[t.ticker] = {
            valorTotal: 0,
            quantidadeTotal: 0,
          }
        if (t.operacao === 'Compra') {
          map[t.ticker].valorTotal += t.valor
          map[t.ticker].quantidadeTotal += t.unidades
          map[t.ticker].valorMedio =
            map[t.ticker].valorTotal / map[t.ticker].quantidadeTotal
        } else if (t.operacao === 'Venda') {
          map[t.ticker].valorTotal -=
            (map[t.ticker].valorTotal * t.unidades) /
            map[t.ticker].quantidadeTotal
          map[t.ticker].quantidadeTotal -= t.unidades
          console.log('lucro total da venda de ' + t.ticker + ": " + -t.valor +  ' ' + t.unidades + ' ' + map[t.ticker].valorMedio)
          t._lucroTotal = -t.valor - t.unidades * map[t.ticker].valorMedio
          t._mes = self.ddmmyyyy2yyyymm(t.data)
          t._produto =
            t.produto !== 'Ação'
              ? t.produto
              : t.ticker.endsWith('34')
              ? 'BDR'
              : t.ticker === 'HASH11' || t.ticker === 'ETHE11'
              ? 'ETF'
              : 'Ação'
          t._produtoSlug = t._produto
            .replace('ç', 'c')
            .replace('ã', 'a')
            .toLowerCase()
        }
        return t
      }

      const transacoes = [...this.realValorTransacoes]

      transacoes.sort((a, b) => {
        const sa = this.ddmmyyyy2yyyymmdd(a.data)
        const sb = this.ddmmyyyy2yyyymmdd(b.data)
        if (sa < sb) return -1
        if (sb < sa) return 1
        return 0
      })

      transacoes.forEach((t) => l.push(fix(t)))
      return l
    },
    vendasPorMes() {
      const l = []
      let mes
      this.vendas.forEach((v) => {
        if (mes !== v._mes) {
          mes = v._mes
          const m = {}
          m.mes = mes
          m.vendas = []
          l.push(m)
        }
        l[l.length - 1].vendas.push(v)
      })
      return l
    },
    vendas() {
      return this.transacoes.filter((t) => t.operacao === 'Venda')
    },
    tabelaDeVendas() {
      const l = []
      const produtos = ['bdr', 'etf', 'acao', 'fii']
      let mesAnterior
      this.vendasPorMes.forEach((m) => {
        const linhas = m.vendas.length

        // Totaliza vendas e lucros por mês
        m.vendas.forEach((venda, idx) => {
          const v = { ...venda }
          if (m['_' + v._produtoSlug + 'Vendas'] === undefined)
            m['_' + v._produtoSlug + 'Vendas'] = 0
          m['_' + v._produtoSlug + 'Vendas'] -= v.valor
          console.log(
            'lucro ' +
              v._produtoSlug +
              ' em ' +
              m.mes +
              ' ' +
              v._lucroTotal +
              ' - total ' +
              m['_' + v._produtoSlug + 'Lucro']
          )
          if (m['_' + v._produtoSlug + 'Lucro'] === undefined)
            m['_' + v._produtoSlug + 'Lucro'] = 0
          m['_' + v._produtoSlug + 'Lucro'] += v._lucroTotal
          console.log(
            'soma ' +
              v._lucroTotal +
              ' - total ' +
              m['_' + v._produtoSlug + 'Lucro']
          )
        })

        // Calcula prejuízo mensal para acumular
        produtos.forEach((p) => {
          // copia a tabela do mês anterior
          m['_' + p + 'Prejuizo'] =
            mesAnterior && mesAnterior['_' + p + 'Prejuizo']
              ? [...mesAnterior['_' + p + 'Prejuizo']]
              : []
          // remove expirados com 12 meses ou mais
          m['_' + p + 'Prejuizo'] = m['_' + p + 'Prejuizo'].filter(
            (i) => this.yyyymmPlus1y(i.mes) > m.mes
          )
          if (m['_' + p + 'Lucro'] < 0) {
            m['_' + p + 'Prejuizo'].push({
              mes: m.mes,
              valor: -m['_' + p + 'Lucro'],
            })
          }
          // ordena os prejuízos pela data para compensarmos nos mais antigos
          m['_' + p + 'Prejuizo'].sort((a, b) => {
            if (a.mes > b.mes) return -1
            else if (a.mes < b.mes) return 1
            else return 0
          })

          // compensa o lucro do mês
          if (m['_' + p + 'Lucro'] > 0) {
            let lucro = m['_' + p + 'Lucro']
            m['_' + p + 'Prejuizo'].forEach((i) => {
              if (i.valor >= lucro) {
                i.valor -= lucro
                lucro = undefined
              } else if (i.valor < lucro) {
                lucro -= i.valor
                i.valor = 0
              }
            })
            if (lucro) m['_' + p + 'LucroDescontado'] = lucro

            // remove acumulos de lucro que foram plenamente utilizados
            m['_' + p + 'Prejuizo'] = m['_' + p + 'Prejuizo'].filter(
              (i) => i.valor > 0
            )
          }

          // soma todos os prejuízos gerando o prejuízo acumulado
          if (m['_' + p + 'Prejuizo'].length)
            m['_' + p + 'PrejuizoAcum'] = m['_' + p + 'Prejuizo'].reduce(
              (previousValue, currentValue) =>
                previousValue + currentValue.valor,
              0
            )
        })

        // Calcula imposto mensal
        produtos.forEach((p) => {
          const aliquota = {
            bdr: 0.15,
            etf: 0.15,
            acao: 0.15,
            fii: 0.2,
          }
          if (m['_' + p + 'LucroDescontado'] > 0) {
            if (p !== 'acao' || m['_' + p + 'Vendas'] > 20000) {
              m['_' + p + 'Imposto'] =
                m['_' + p + 'LucroDescontado'] * aliquota[p]
              if (m._impostoDevido === undefined) m._impostoDevido = 0
              m._impostoDevido = m._impostoDevido + m['_' + p + 'Imposto']
            }
          }
        })

        // acumula impostos
        m._impostoAcum = 0
        if (m._impostoDevido) m._impostoAcum += m._impostoDevido
        if (mesAnterior && mesAnterior._impostoAcum)
          m._impostoAcum += mesAnterior._impostoAcum
        if (m._impostoAcum > 10) {
          m._impostoAPagar = m._impostoAcum
          m._impostoAcum = undefined
        }

        // Aplica o resultado do cálculo mensal na primeira linha de vendas do mês
        m.vendas.forEach((venda, idx) => {
          const v = { ...venda }
          if (idx === 0) {
            v.rowSpan = linhas
            produtos.forEach((p) => {
              v['_' + p + 'Vendas'] = m['_' + p + 'Vendas']
              v['_' + p + 'Lucro'] = m['_' + p + 'Lucro']
              v['_' + p + 'LucroDescontado'] = m['_' + p + 'LucroDescontado']
              v['_' + p + 'Prejuizo'] = m['_' + p + 'Prejuizo']
              v['_' + p + 'PrejuizoAcum'] = m['_' + p + 'PrejuizoAcum']
              v['_' + p + 'Imposto'] = m['_' + p + 'Imposto']
            })
            v._impostoDevido = m._impostoDevido
            v._impostoAcum = m._impostoAcum
            v._impostoAPagar = m._impostoAPagar
          }
          l.push(v)
        })
        mesAnterior = m
      })
      return l
    },
  },
  methods: {
    ddmmyyyy2yyyymmdd(data) {
      const dia = data.split('/')[0]
      const mes = data.split('/')[1]
      const ano = data.split('/')[2]

      return ano + '-' + ('0' + mes).slice(-2) + '-' + ('0' + dia).slice(-2)
    },
    ddmmyyyy2yyyymm(data) {
      const mes = data.split('/')[1]
      const ano = data.split('/')[2]

      return ano + '-' + ('0' + mes).slice(-2)
    },
    yyyymmPlus1y(data) {
      const ano = data.split('-')[0]
      const mes = data.split('-')[1]

      return ano + 1 + '-' + ('0' + mes).slice(-2)
    },
    formatPrice(value) {
      if (!value) return
      const val = (value / 1).toFixed(2).replace('.', ',')
      return val.toString().replace(/\B(?=(\d{3})+(?!\d))/g, '.')
    },
  },
}
</script>
<style>
td,
th {
  font-size: 70%;
}
th {
  white-space: nowrap;
  text-align: right;
}
td {
  text-align: right;
}
td.title {
  text-align: left;
}
th.title {
  text-align: left;
}
h1 {
  font-size: 150%;
}
.text-muted {
  font-size: 80%;
}
</style>
