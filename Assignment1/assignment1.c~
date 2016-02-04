#include "contiki.h"
#include "net/rime.h"
#include "dev/cc2420.h"
#include "dev/cc2420_const.h"
#include "dev/spi.h"
#include "dev/leds.h"

#include <stdio.h>
#include <string.h>

int PACKETS_RECVD = 0;
int PACKETS_SENT = 0;
int i = 0;

/*---------------------------------------------------------------------------*/
PROCESS(example_unicast_process, "Example unicast");
AUTOSTART_PROCESSES(&example_unicast_process);
/*---------------------------------------------------------------------------*/

static struct unicast_conn uc;

static void
unicast_prr(int k)
{
  if(k==1)
  {
    PACKETS_SENT++;
    printf("No of packets sent: %d\n", PACKETS_SENT);
  }
  if(k==2)
  {
    PACKETS_RECVD++;
    printf("No of packets received: %d\n", PACKETS_RECVD);
  }
}

static void
unicast_msgsend()
{
  rimeaddr_t addr;
    
  char seq[20];
  sprintf(seq, "Sender Check %d", i);
  packetbuf_copyfrom(seq, 20);
  addr.u8[0] = 3;
  addr.u8[1] = 0;

  if(!rimeaddr_cmp(&addr, &rimeaddr_node_addr))
  {
    unicast_send(&uc, &addr);
    unicast_prr(1);
    printf("Sender: Packet Sent\n");
    i++;
  }
}

static void
unicast_ack(const rimeaddr_t *ackto, char *no)
{
  char ack[20];
  sprintf(ack, "Ack%s", no);
  packetbuf_copyfrom(ack, 20);
  unicast_send(&uc, ackto);
  unicast_prr(2);
  printf("Base Station: Acknowledgment sent \n");
}


static void
recv_uc(struct unicast_conn *c, const rimeaddr_t *from)
{
  printf("Unicast message received from %d.%d : '%s'\n", from->u8[0], from->u8[1], (char *)packetbuf_dataptr());
  printf("The Received Signal Strength is: %d\n", (cc2420_last_rssi -45) );

  char *recvdmsg = (char *)packetbuf_dataptr();
  char *no = strchr(recvdmsg, ' ');  
  leds_toggle(LEDS_ALL);

  if(from->u8[0] == 1)
  {
    unicast_ack(from , no);
  }
}
static const struct unicast_callbacks unicast_callbacks = {recv_uc};

/*---------------------------------------------------------------------------*/
PROCESS_THREAD(example_unicast_process, ev, data)
{
  PROCESS_EXITHANDLER(unicast_close(&uc);)
    
  PROCESS_BEGIN();

  unicast_open(&uc, 26, &unicast_callbacks);
  cc2420_on();

  while(1) {
    static struct etimer et;
    
    etimer_set(&et, CLOCK_SECOND*2);
    
    PROCESS_WAIT_EVENT_UNTIL(etimer_expired(&et));
    unicast_msgsend();
  }

  PROCESS_END();
}
/*---------------------------------------------------------------------------*/
